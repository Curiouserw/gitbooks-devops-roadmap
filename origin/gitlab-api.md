# Gitlab Restful API

# 一、简介

GItlab拥有Restful的API接口来操作其中的资源对象。

Gitlab Restful API Docs：https://docs.gitlab.com/ee/api/README.html#status-codes

## 1、接口请求方式

- Endpoint：`http://gitlab地址/api/api版本/资源对象/操作URI?参数1&参数2`

- 目前接口版本为V4，V3在11.0版本后已经移除

- 接口需要认证，支持的验证方式：

  - [OAuth2 tokens](https://docs.gitlab.com/ee/api/README.html#oauth2-tokens)

    ```bash
    curl --header "Authorization: Bearer OAUTH-TOKEN" "https://gitlab.example.com/api/v4/projects"
    # 或者
    curl "https://gitlab.example.com/api/v4/projects?access_token=OAUTH-TOKEN"
    ```

  - [Personal access tokens](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)

    ```bash
    curl --header "PRIVATE-TOKEN: <your_access_token>" "https://gitlab.example.com/api/v4/projects"
    # 或者
    curl "https://gitlab.example.com/api/v4/projects?private_token=<your_access_token>"
    # 或者
    curl --header "Authorization: Bearer <your_access_token>" "https://gitlab.example.com/api/v4/projects"
    ```

  - [Project access tokens](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html)

    ```bash
    curl --header "PRIVATE-TOKEN: <your_access_token>" "https://gitlab.example.com/api/v4/projects"
    # 或者
    curl "https://gitlab.example.com/api/v4/projects?private_token=<your_access_token>"
    # 或者
    curl --header "Authorization: Bearer <your_access_token>" "https://gitlab.example.com/api/v4/projects"
    ```

  - [Session cookie](https://docs.gitlab.com/ee/api/README.html#session-cookie)

  - [GitLab CI/CD job token](https://docs.gitlab.com/ee/api/README.html#gitlab-ci-job-token)

- 接口返回的是JSON数据格式（在Linux下可以使用jq进行处理）

  ```bash
  curl --header "PRIVATE-TOKEN: <your_access_token>" \
  "https://gitlab.example.com/api/v4/projects" | jq -r '.[].id'
  ```

- 如果某些资源对象的URL操作中包含特殊字符，需要做URL编码处理

  例如：资源对象路径包含的`/`可以使用`%2F`进行处理

  ```bash
  GET /api/v4/projects/1/repository/files/src%2FREADME.md?ref=master
  GET /api/v4/projects/1/branches/my%2Fbranch/commits
  GET /api/v4/projects/1/repository/tags/my%2Ftag
  ```

- 请求资源对象参数中的数组和hash

  - 数组

    ```bash
    curl --request POST --header "PRIVATE-TOKEN: <your_access_token>" \
    -d "import_sources[]=github" \
    -d "import_sources[]=bitbucket" \
    "https://gitlab.example.com/api/v4/some_endpoint"
    ```

  - Hash

    ```bash
    curl --request POST --header "PRIVATE-TOKEN: <your_access_token>" \
    --form "namespace=email" \
    --form "path=impapi" \
    --form "file=@/path/to/somefile.txt"
    --form "override_params[visibility]=private" \
    --form "override_params[some_other_param]=some_value" \
    "https://gitlab.example.com/api/v4/projects/import"
    ```

  - Hash数组

    ```bash
    curl --globoff --request POST --header "PRIVATE-TOKEN: <your_access_token>" \
    "https://gitlab.example.com/api/v4/projects/169/pipeline?ref=master&variables[][key]=VAR1&variables[][value]=hello&variables[][key]=VAR2&variables[][value]=world"
    
    curl --request POST --header "PRIVATE-TOKEN: <your_access_token>" \
    --header "Content-Type: application/json" \
    --data '{ "ref": "master", "variables": [ {"key": "VAR1", "value": "hello"}, {"key": "VAR2", "value": "world"} ] }' \
    "https://gitlab.example.com/api/v4/projects/169/pipeline"
    ```

- 请求分页

  对于一些有大量返回数据的资源请求，gitlab为了保持性能，是有默认分页操作的。支持的分页方式

  - 基于`Offset`的分页(所有接口都支持)

    - 参数

      | Parameter  | Description                                 |
      | :--------- | :------------------------------------------ |
      | `page`     | 页数，默认为1                               |
      | `per_page` | 一页显示的对象个数 (默认: `20`,最大: `100`) |

      ```bash
      curl --request PUT --header "PRIVATE-TOKEN: <your_access_token>" "https://gitlab.example.com/api/v4/namespaces?per_page=50"
      ```

    - 分页返回的Header

      ```bash
      curl --head --header "PRIVATE-TOKEN: <your_access_token>" "https://gitlab.example.com/api/v4/projects/9/issues/8/notes?per_page=3&page=2"
      
      HTTP/2 200 OK
      cache-control: no-cache
      content-length: 1103
      content-type: application/json
      date: Mon, 18 Jan 2016 09:43:18 GMT
      link: <https://gitlab.example.com/api/v4/projects/8/issues/8/notes?page=1&per_page=3>; rel="prev", <https://gitlab.example.com/api/v4/projects/8/issues/8/notes?page=3&per_page=3>; rel="next", <https://gitlab.example.com/api/v4/projects/8/issues/8/notes?page=1&per_page=3>; rel="first", <https://gitlab.example.com/api/v4/projects/8/issues/8/notes?page=3&per_page=3>; rel="last"
      status: 200 OK
      vary: Origin
      x-next-page: 3
      x-page: 2
      x-per-page: 3
      x-prev-page: 1
      x-request-id: 732ad4ee-9870-4866-a199-a9db0cde3c86
      x-runtime: 0.108688
      x-total: 8
      x-total-pages: 3
      ```

      | Header          | Description    |
      | :-------------- | :------------- |
      | `x-next-page`   | 下一页的索引   |
      | `x-page`        | 当前页的索引   |
      | `x-per-page`    | 每页的对象个数 |
      | `X-prev-page`   | 上一页的索引   |
      | `x-total`       | 所有的对象格式 |
      | `x-total-pages` | 所有页的个数   |

  - 基于`Keyset`的分页(仅支持部分接口 )

    - 参数

      | Parameter    | Description                                 |
      | :----------- | :------------------------------------------ |
      | `pagination` | `keyset` (开启分页使用keyset).              |
      | `per_page`   | 一页显示的对象个数 (默认: `20`,最大: `100`) |

    - 分页返回的Header

      ```bash
      curl --request GET --header "PRIVATE-TOKEN: <your_access_token>" "https://gitlab.example.com/api/v4/projects?pagination=keyset&per_page=50&order_by=id&sort=asc"
      
      HTTP/1.1 200 OK
          ...
          Links: <https://gitlab.example.com/api/v4/projects?pagination=keyset&per_page=50&order_by=id&sort=asc&id_after=42>; rel="next"
          Link: <https://gitlab.example.com/api/v4/projects?pagination=keyset&per_page=50&order_by=id&sort=asc&id_after=42>; rel="next"
          Status: 200 OK
          ...
      ```

- 接口返回状态码

  | 状态返回码               | 描述                                                         |
  | :----------------------- | :----------------------------------------------------------- |
  | `200 OK`                 | `GET`,`PUT `或者`DELETE`请求已处理完成，已返回JSON结构的数据 |
  | `204 No Content`         | 接口已处理完成了请求，但是没有什么数据可返回的               |
  | `201 Created`            | `POST` 请求已处理完成，已返回JSON结构的数据                  |
  | `304 Not Modified`       | 自上次请求以来，该资源尚未修改。                             |
  | `400 Bad Request`        | 请求参数不完整                                               |
  | `401 Unauthorized`       | 用户未认证，请求需要认证Token                                |
  | `403 Forbidden`          | 请求中认证Token没有相应的权限对请求中的对象进行操作          |
  | `404 Not Found`          | 请求操作的资源没发现。可能是参数不对，没有匹配到准确的对象   |
  | `405 Method Not Allowed` | 请求中的资源对象不支持该请求方法                             |
  | `409 Conflict`           | 请求的资源操作跟已有的现有对象有冲突。                       |
  | `412`                    | 请求的资源操作被拒绝啦。如果尝试删除资源时提供了`If-Unmodified-Since`标头，则可能会发生这种情况。 |
  | `422 Unprocessable`      | The entity couldn’t be processed.                            |
  | `429 Too Many Requests`  | 请求被限制了！                                               |
  | `500 Server Error`       | 在服务端处理请求过程中出现了错误                             |


# 二、Project API

## DQL

### 1、列出所有仓库

**Endpoint**

> GET /projects

**文档**

https://docs.gitlab.com/ee/api/projects.html#list-all-projects

**参数**

| 参数                          | 类型     | 是否必须 | 描述                                                         |
| :---------------------------- | :------- | :------- | :----------------------------------------------------------- |
| `archived`                    | boolean  | No       | 是否列出`archived`状态的                                     |
| `id_after`                    | integer  | No       | Limit results to projects with IDs greater than the specified ID. |
| `id_before`                   | integer  | No       | Limit results to projects with IDs less than the specified ID. |
| `last_activity_after`         | datetime | No       | Limit results to projects with last_activity after specified time. Format: ISO 8601 YYYY-MM-DDTHH:MM:SSZ |
| `last_activity_before`        | datetime | No       | Limit results to projects with last_activity before specified time. Format: ISO 8601 YYYY-MM-DDTHH:MM:SSZ |
| `membership`                  | boolean  | No       | Limit by projects that the current user is a member of.      |
| `min_access_level`            | integer  | No       | Limit by current user minimal [access level](https://docs.gitlab.com/ee/api/members.html#valid-access-levels). |
| `order_by`                    | string   | No       | Return projects ordered by `id`, `name`, `path`, `created_at`, `updated_at`, or `last_activity_at` fields. `repository_size`, `storage_size`, `packages_size` or `wiki_size` fields are only allowed for admins. Default is `created_at`. |
| `owned`                       | boolean  | No       | Limit by projects explicitly owned by the current user.      |
| `repository_checksum_failed`  | boolean  | No       | Limit projects where the repository checksum calculation has failed ([Introduced](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/6137) in [GitLab Premium](https://about.gitlab.com/pricing/) 11.2). |
| `repository_storage`          | string   | No       | Limit results to projects stored on `repository_storage`. *(admins only)* |
| `search_namespaces`           | boolean  | No       | Include ancestor namespaces when matching search criteria. Default is `false`. |
| `search`                      | string   | No       | Return list of projects matching the search criteria.        |
| `simple`                      | boolean  | No       | Return only limited fields for each project. This is a no-op without authentication as then *only* simple fields are returned. |
| `sort`                        | string   | No       | Return projects sorted in `asc` or `desc` order. Default is `desc`. |
| `starred`                     | boolean  | No       | Limit by projects starred by the current user.               |
| `statistics`                  | boolean  | No       | Include project statistics.                                  |
| `visibility`                  | string   | No       | Limit by visibility `public`, `internal`, or `private`.      |
| `wiki_checksum_failed`        | boolean  | No       | Limit projects where the wiki checksum calculation has failed ([Introduced](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/6137) in [GitLab Premium](https://about.gitlab.com/pricing/) 11.2). |
| `with_custom_attributes`      | boolean  | No       | Include [custom attributes](https://docs.gitlab.com/ee/api/custom_attributes.html) in response. *(admins only)* |
| `with_issues_enabled`         | boolean  | No       | Limit by enabled issues feature.                             |
| `with_merge_requests_enabled` | boolean  | No       | Limit by enabled merge requests feature.                     |
| `with_programming_language`   | string   | No       | Limit by projects which use the given programming language.  |

**示例**

```bash
GET /projects?custom_attributes[key]=value&custom_attributes[other_key]=other_value
```

