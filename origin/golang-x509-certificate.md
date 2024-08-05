# Go生成x509相关证书

# 一、简介







# 二、证书工具类

### ①保存证书到文件

```go
func saveCertificateToFile(certBytes []byte, certType, filePath string, perm os.FileMode) error {
	err := os.MkdirAll(filepath.Dir(configrootpath+filePath), 0755)
	if err != nil {
		fmt.Printf("failed to create directory: %v", err)
	}
	certPEM := pem.EncodeToMemory(&pem.Block{Type: certType, Bytes: certBytes})
	err = os.WriteFile(configrootpath+filePath, certPEM, perm)
	if err != nil {
		return err
	}
	return nil
}
```

# 三、生成自签名CA证书

```go
func generateCACertificate(country, province, city string) []byte {
	caKey, err := rsa.GenerateKey(rand.Reader, 4096)
	if err != nil {
		log.Fatal(err)
	}

	// 创建CA证书
	caCert := &x509.Certificate{
		SerialNumber: big.NewInt(1),
		Subject:      pkix.Name{CommonName: "test.top"},
		NotBefore:    time.Now(),
		NotAfter:     time.Now().AddDate(100, 0, 0),
		IsCA:         true,
		KeyUsage:     x509.KeyUsageDigitalSignature | x509.KeyUsageCRLSign | x509.KeyUsageCertSign,
		// 基本约束
		BasicConstraintsValid: true,
		// 证书最大路径，该级 CA 能签发子 CA 的证书链路径深度
		MaxPathLen: 0,
		// 不允许该 CA 签发子 CA
		MaxPathLenZero: false,
		// OCSP 响应服务器
		OCSPServer: nil,
		// CRL 分发点
		CRLDistributionPoints: nil,
	}

	// CA扩展配置
	caCert.SubjectKeyId = []byte{1, 2, 3, 4, 5}
	caCert.AuthorityKeyId = []byte{1, 2, 3, 4, 5}

	// 生成CA证书
	caCertBytes, err := x509.CreateCertificate(rand.Reader, caCert, caCert, caKey.Public(), caKey)
	if err != nil {
		log.Fatal(err)
	}

	// 保存CA证书到文件
	err = saveCertificateToFile(caCertBytes, "CERTIFICATE", "pki/ca.crt", 0644)
	if err != nil {
		log.Fatal(err)
	}
	// 保存CA私钥到文件
	caKeyPEMBytes := x509.MarshalPKCS1PrivateKey(caKey)
	err = saveCertificateToFile(caKeyPEMBytes, "RSA PRIVATE KEY", "pki/ca.key", 0600)
	if err != nil {
		log.Fatal(err)
	}
	return caCertBytes
}
```



# 四、自签CA证书签署服务端证书

```bash
func generateServerCertificate(serverName string) {
	caCert, caKey := getCACertificateAndKey()

	serverKey, err := rsa.GenerateKey(rand.Reader, 4096)
	if err != nil {
		log.Fatal(err)
	}
	serverCert := &x509.Certificate{
		SerialNumber:          big.NewInt(2),
		Subject:               pkix.Name{CommonName: serverName},
		NotBefore:             time.Now(),
		NotAfter:              time.Now().AddDate(1, 0, 0),
		KeyUsage:              x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature,
		ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},
		BasicConstraintsValid: true,
	}
	serverCertBytes, err := x509.CreateCertificate(rand.Reader, serverCert, caCert, serverKey.Public(), caKey)
	if err != nil {
		log.Fatal(err)
	}

	// 保存服务端证书到文件
	err = saveCertificateToFile(serverCertBytes, "CERTIFICATE", "pki/server.crt", 0644)
	if err != nil {
		log.Fatal(err)
	}

	// 保存服务端私钥到文件
	serverKeyPEMBytes, err := x509.MarshalPKCS8PrivateKey(serverKey)
	if err != nil {
		log.Fatal(err)
	}
	err = saveCertificateToFile(serverKeyPEMBytes, "RSA PRIVATE KEY", "pki/server.key", 0600)
	if err != nil {
		log.Fatal(err)
	}
}
```



# 五、自签CA证书签署客户端证书

```go
func generateClientCertificate(clientName string) {
	caCert, caKey := getCACertificateAndKey()

	serialNumber, _ := rand.Int(rand.Reader, new(big.Int).Lsh(big.NewInt(1), 128))

	// 生成客户端证书私钥
	privateKey, _ := ecdsa.GenerateKey(elliptic.P384(), rand.Reader)
	notBefore := time.Now()
	notAfter := notBefore.AddDate(50, 0, 0)
	clientCertTemplate := x509.Certificate{
		SerialNumber:          serialNumber,
		Subject:               pkix.Name{CommonName: clientName},
		NotBefore:             notBefore,
		NotAfter:              notAfter,
		KeyUsage:              x509.KeyUsageDataEncipherment | x509.KeyUsageDigitalSignature,
		ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageClientAuth},
		BasicConstraintsValid: true,
	}
	clientCertBytes, err := x509.CreateCertificate(rand.Reader, &clientCertTemplate, caCert, &privateKey.PublicKey, caKey)
	if err != nil {
		log.Fatal(err)
	}
	// 保存客户端证书和证书私钥到文件
	saveCertificateToFile(clientCertBytes, "CERTIFICATE", "pki/clients/"+clientName+".crt", 0644)
	clientKeyPEMBytes, _ := x509.MarshalECPrivateKey(privateKey)
	saveCertificateToFile(clientKeyPEMBytes, "EC PRIVATE KEY", "pki/clients/"+clientName+".key", 0600)
	indexFile, err := os.OpenFile(configrootpath+"pki/index.txt", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		log.Fatal(err)
	}
	defer indexFile.Close()

	serialNumberHex := fmt.Sprintf("%X", clientCertTemplate.SerialNumber)
	clientCertString := "V " + serialNumberHex + " " + clientCertTemplate.NotBefore.Format("060102150405Z") + " " + clientCertTemplate.NotAfter.Format("060102150405Z") + " unknown /CN=" + clientName + "\n"
	if _, err := indexFile.WriteString(clientCertString); err != nil {
		log.Fatal(err)
	}

	clientConfig := readFile(configrootpath + "common-client-config.txt")

	// 在变量中追加内容
	additionalContent := "<cert>\n" + string(pem.EncodeToMemory(&pem.Block{Type: "CERTIFICATE", Bytes: clientCertBytes})) + "</cert>\n<key>\n" + string(pem.EncodeToMemory(&pem.Block{Type: "EC PRIVATE KEY", Bytes: clientKeyPEMBytes})) + "</key>\n"
	clientConfig = append(clientConfig, additionalContent...)
	os.WriteFile(configrootpath+"../client/"+clientName+".ovpn", clientConfig, 0644)
}
```



# 六、验证证书









# 参考：

- https://blog.yeziruo.com/archives/148.html