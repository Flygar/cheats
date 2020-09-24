### 交叉编译
```sh
# The -v flag causes tidy to print information about removed modules to standard error.
go mod tidy [-v]

# 交叉编译: 指定目标操作系统为linux; 目标操作系统的架构为x86_64; 编译时给源代码的变量传入参数;
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -v -a -mod=vendor -tags netgo -ldflags "-X github.com/flygar/project/pkg/pluginlib/utils.Version=$(VERSION) -X github.com/flygar/project/pkg/pluginlib/utils.deployment=$(DEPLOYMENT)" -o ./bin/app ./cmd/agent
```