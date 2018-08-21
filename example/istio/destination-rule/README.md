test:
curl http://n1.ng.com/ -H "user: test"

https://blog.csdn.net/bytxl/article/details/46987137

    - headers:
        user:
          exact: test

    - headers:
        cookie:
          regex: "^(.*?;)?(email=[^;]*@some-company-name.com)(;.*)?$"
