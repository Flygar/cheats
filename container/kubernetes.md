
```sh
# 查看指定 namespace 下的 pod 日志
kubectl logs --tail 200 -n <namespace> <podName>

# json格式化指定 namespace 下的 pod 信息，输出 status 部分
kubectl get pods -n <namespace> <podName> -o json | jq .status
```

## helm
```sh
# 创建chart
helm create mychart
rm -rf mychart/templates/*.*

# 测试 --set用于覆盖values.yaml文件中的变量; -n 指定<namespace>, 用于{{.Release.Namespace}} 
helm install --dry-run --debug --generate-name -n default --set favoriteDrink=slurm ./mychart

# 查看release 
helm get manifest <.Release.Name>
# 删除release
helm delete <.Release.Name>

```

## 示例
```yaml
# values.yaml 文件
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
```
Helm 拥有超过 60 种可用函数。
```yaml
# templates/configmap1.yaml 文件
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap                         # 可以使用 --generate-name 随机生成
data:
  myvalue: "Hello World"          
  food: {{.Values.favorite.food | upper | quote}}             # 使用管道将values.yaml文件中的值传给upper函数计算后再传给quote函数计算
  drink: {{.Values.favorite.drink | repeat 5 | quote}}        # repeat函数接受2个参数，将值重复5次
  drink: {{.Values.favorite.drink | default "tea" | quote}}   # 指定默认值“tea”, 防止在values.yaml中被省略
  drink: {{.Values.favorite.drink | default (printf "%s-tea" (include "fullname" .))}}  # default 常用来计算 
  {{- if eq .Values.favorite.drink "coffee"}}                 # 如果values.yaml中包含 favorite.drink:coffee, 则输出应该包含 mug: true 标志. {{- : 删除左侧空格
  mug: true
  {{- end}}
  {{- with .Values.favorite}}                                 # 使用with改变 "." 的作用范围。当前使得 "." 指向".Values.favorite"
  drink: {{.drink | default "tea" | quote}}
  food: {{.food | upper | quote}}
  {{- end}}
  toppings: |-                                                # 使用range，也能改变 "." 的作用范围; YAML 中的 |- 标记表示一个多行字符串。
    {{- range .Values.pizzaToppings}}
    - {{. | title | quote}}                               
    {{- end}}
  sizes: |-                                                   # 使用 tuple 在模版中创建一个列表
    {{- range tuple "small" "medium" "large"}}
    - {{.}}
    {{- end}}
  {{- $relname := .Release.Name -}}                           # 可以在range或with外部定义变量, 内部使用
  {{- with .Values.favorite}}
  drink: {{.drink | default "tea" | quote}}
  food: {{.food | upper | quote}}
  release: {{$relname}}                                       # 内部使用变量(不会出错)，使用{{.Release.Name}}会报错
  {{- end}}
  toppings: |-                                                # 在range中定义变量，同时捕获索引和值
    {{- range $index, $topping := .Values.pizzaToppings}}
      {{$index}}: {{ $topping }}
    {{- end}}
# {{if and .Values.fooString (eq .Values.fooString "foo") }}
# {{if or .Values.anUnsetVariable (not .Values.aSetVariable) }}

```
### 模版的声明和使用  
建议使用 include+indent 代替template  
```yaml
# templates/_helpers.tpl 文件
{{/* 
Generate basic labels                         # 模版描述，相当于注释
*/}}               
{{- define "mychart.labels" }}                # 声明模版，通常在 _helpers.tpl 文件中声明模版
labels:
  generator: helm
  date: {{ now | htmlDate }}
  chart: {{ .Chart.Name }}                  # chart.yml文件中的变量，和values.yaml类似。不过变量首字母要大写
  version: {{ .Chart.Version }}
{{- end }}

{{- define "mychart.app" }}                  # 声明模版
app_name: {{ .Chart.Name }}
app_version: "{{ .Chart.Version }}+{{ .Release.Time.Seconds }}"
{{- end }}
---
# templates/configmap2.yaml 文件
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" .}}            # 使用模版，直接引用 _helpers.tpl 文件中的模版变量; 注意末尾的 ".",表示顶级范围，也可以传入 .Values 或者 .Values.favorite 等等。 这个范围会被 _helpers.tpl 文件中使用到。
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
{{- include "mychart.app" . | indent 2 }}       # 使用管道调用函数来缩进，建议使用这种方法代替template
```

### 访问文件
Helm 通过 .Files 对象提供对文件的访问
```yaml
# 主目录下创建下面3个文件
# config1.toml
message = Hello from config 1
# config2.toml
message = Hello from config 2
# config3.toml
message = Hello from config 3

# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  {{- $files := .Files}}
  {{- range tuple "config1.toml" "config2.toml" "config3.toml"}}
  {{.}}: |-                                            #  通过 .Files.Get 获得文件内容
    {{$files.Get .}}                               
  {{- end}}
  token: |-                                            #  对config1.toml文件内容进行base64 encode
    {{.Files.Get "config1.toml" | b64enc}}          
  some-file.txt: |-                                    #  访问文件中的每一行
    {{- range .Files.Lines "foo/bar.txt"}}
    {{.}}
    {{- end }}

# templates/configmap2.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: conf
data:
  {{- (.Files.Glob "foo/*").AsConfig | nindent 2 }}     # 使用Glob配合AsConfig工具函数生成ConfigMap数据
---
apiVersion: v1
kind: Secret
metadata:
  name: {{.Release.Name}}-secret
type: Opaque
data:
  {{(.Files.Glob "bar/*").AsSecrets | nindent 2 }}       # 使用Glob配合AsSecrets工具函数生成secret数据

```

### NOTES.txt 文件
安装说明添加到 chart，只需创建一个 templates/NOTES.txt 文件即可
- 和一个模板一样处理，并且具有所有可用的普通模板函数和对象。
```txt
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get {{ .Release.Name }}
```
