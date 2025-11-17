# Shiro 序列化数据解析还原

## 功能概述
- 对 Apache Shiro 加密的序列化数据进行解密、解析，并尝试生成可读的 Java 伪代码，便于阅读分析。
- 形式：Web 服务（HTTP），默认监听 `8080` 端口；支持通过启动`-listen`参数指定监听地址与端口。

## 功能列表
- 解密
  - 支持指定密钥（Base64）的解密尝试。
  - 支持默认密钥列表的批量尝试解密。
  - 自动识别 AES-CBC/AES-GCM 并校验序列化数据有效性。
- 生成
  - 生成可读 Java 伪代码。

## 接口设计
- 基本信息：
  - 默认监听：`:8080`
  - 启动参数：`-listen  <ip:port>` 指定监听地址与端口，例如 `-listen 192.168.100.200:8888`
- 访问方式：
  - 浏览器访问监听端口，输入请求数据尝试解密
<img src="https://github.com/chuanjiesun/shiro_rememberMe_decrypt/blob/master/shiro_input.png" width="800" alt="">
<img src="https://github.com/chuanjiesun/shiro_rememberMe_decrypt/blob/master/shiro_output.png" width="800" alt="">
<img src="https://github.com/chuanjiesun/shiro_rememberMe_decrypt/blob/master/shiro_output2.png" width="800" alt="">

也可以通过接口直接发送数据：
- 解密接口：`POST /shiro_decrypt`
- 请求内容：
  - JSON：`{"key":"", "encrypt_data":"<ShiroCookieBase64>"}`
  - 字段说明：
    - `key`（可选）：Base64 密钥；非空时仅使用该密钥尝试解密；为空则尝试默认密钥列表。
    - `encrypt_data`（必填）：Shiro Cookie 的 Base64 字符串
- 响应内容：
  - 结构：
    ```json
    {
      "error": "空为成功；非空为错误详情",
      "data": {
        "key": "使用的Base64密钥",
        "iv": "IV/Nonce（Base64）",
        "aes_type": "AES_CBC 或 AES_GCM",
        "decrypt_data": "明文序列化数据（Base64）"
      },
      "java_code": "Java伪代码文本"
    }
    ```
  - 说明：
    - 成功时：`error` 为空；`java_code` 为伪代码；`data` 内包含用于参考的 `key/iv/aes_type/decrypt_data`。
    - 失败时：`error` 为错误详情；`java_code` 为空；`data` 字段可能为空或各项为空。

## 示例请求
- 使用 `curl`（POST JSON）：
  - `curl -H "Content-Type: application/json" localhost:8080/shiro_decrypt --data '{"key":"", "encrypt_data":"<Base64Cookie>"}'`


## 输入与输出
- 输入（HTTP 请求）：
  - `POST /shiro_decrypt`，JSON 包含 `key`（可选）与 `encrypt_data`（必填）。
- 输出（HTTP 响应）：
  - JSON 包含 `error`、`data`（元信息）与 `java_code`（伪代码）。

```bash
curl localhost:8080/shiro_decrypt --data "{\"key\":\"\", \"encrypt_data\":\"j89Q3Fk+abCUmRrmwYJkgRh2qs6D59S6ShuCE6tauEJCJBtiPYBBcfXsdUg9pB8f4vPO5CHU5o3BtIo+kskUaGBmrAkNktypEacuyGm0EacrUKYguCMNThUI8cM+GbneobtXGYXhxZCn4O4gJHKzk7bq3w+dzrmuuMOuaISVTLmZJ1JCn52FdWYPVC+V2uuyDdXzvaU8GkFvsE05Ln6YmooYMDpVdQ73mRqu2SjikpmrSeNDeA4KnWX8DrEe9SVRN4kudMabmPjBYdUV0dFzhW5lcwFW/+gRvbKWc/EXdh9zEv+VHjmR1NfNWKsVF2Ydpc//LZJb/xMaWtdSTqYly7STNbATfJoFcTem9aeJknuiRGDGlA54K+9Fzq8HaqWB+lSjaZjiS+PKaAndqGp2SY6JWchtt9wpn9yLDkQorOBjsRg6jvppaSzUtIl82TbdbZL7UicVovEUg2g+L1TEo9MYwASgOPsjYA7cKoyCpVdiuXWax3zQnpcONfmD7RorbmgU2IcrENFuE88thRvTZRgDOD/5LFGqWiWf33tSsIr9NQflFgoeXzSHYXngMT+NaeD29Z0mVEMibiOSLqNCCrIe1QBe5XUv2tf0/RfAavnP3UF2l1CFaFwohvCI2+j73nsfGWL6+02m5DwDvitrN1vcdKRbkhJY7utwBJmfHFmz/FZsSrbdWGqUcKtrcRw40wk0Vd/vPhgn3HziCLtM1cVovI3JzzhI1HYabGFgGfGgmr5CEv+wd28ch6GCrV4XwB/QryVNhbWGBkj1W2YiTcJdrqHURJAdx4x66oRYCqvOeVwmhgdRq98QazYp1/0jmp2xceuXKpo1TFyKGpgFJyScuDMFD46eDwnNtoztTYOOlNT67yO4rPD2W9C/RQwpodBBfW+e7oO2f7HNIrbTQS2acz9/dD1bD6+WXBwKpUcXcJnqkw18dmcWJyLhyZt7STl510ljSpDFPJIBXlilgxqg7wU2oh7XeGSiq1L+xssl2LUpYdlYK1At9cDT86/1GWvOxVVmWXZ+0f38xfQHQRJ4XIa1SL4DuixZYQac1O8ePI9+94qI9shAdPs41wIckugk/7JKovvNzSa1UsOrst2+ZzntSS1c1OuQsNWFuzqwgRBwtdqVjwvoKGYWzMgzBwSnnvMdsisBNSBdZKmc+WDpJsjZwudbZMZu+Y5SCV4XOMC0e9cs4AT0pHZ/qH1UK/wtFl1GmENS8u6WRD0AOGDSM80K+99R5/cKfAle2qbBsLE01Iykksm1Q+lVETzWhIWRram9Yk/BwPUynQtbRAZZio2ZnYzwx8QLy1t0PwPNE+k9nexa/vnlPbmNF9jcG9TLfCVhd232HXEs6tVqRqXpNfLzd9898H5r1PCAyKcrQP4C0JXQv8XnvJ4bfBU8sFnMDgXxnUXfzrLfkKHynuEGejS9FTSZqGQrNRrC8FBzDE80sI8eXs7EW5qvU1pXamjH8Py0uhukgTpzph8uFA0suUPbxysCXegwncrRTP+Ql1SmC8aehAeJxoDUv7MvUrqQBdY0CpOVny/nZL7OKJODE1ANy89/Z1FpFprq5ctPjUEKvLkzqBmlydDCqg2/Smcm6Y3twDb3yCE4Z2H5Q+TqtEk35gyZLURVFoO8qy70x1ozyVkHWPWOklroP6OrNgjmUQY2PWYJ+DkRbdWNqgkTUlAN4i7mVOw7B6cgExw1hYRc0/s0hrqiHVnEIHfzvzKNJ8+P0mcSgSzLQ5yZf8HLsCZ0cx9uQkwXis0SbXblZEsd1C2cCJ0dDdeJF8J9GBeWkW7GRFvqBaqIbe9htWsjkQRlDtoV0oCzLY8tnzykAUXK9XWMjI7CbTy6c0UMlJLzURek5fE5+sljcyVH/uplQJT0zlvfLLpSSS7X6pzmYMiEuAvxmEWYgTEDkeQc\"}"
```

## 在线站点
搭建一个简易在线站点：[https://shiro.tools-trunk.top/](https://shiro.tools-trunk.top/)
