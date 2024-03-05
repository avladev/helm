# Install HELM
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

# Install HELM unit tests
```bash
helm plugin install https://github.com/helm-unittest/helm-unittest.git
```

# Structure
Templates = Manifests
Templates + Values = Manifests
```txt
| - templates/    # Holds the templates which will be converted to k8s manifests.
| - charts/       # Holds subcharts and dependencies for the top level Helm Chart.
| - Chart.yaml    # Metadata for the chart and a list of dependencies.
| - values.yaml   # Holds configuration values
| - .helmignore   # Ignore files from Helm process
| - NOTES.txt     # Bla bla (It is also a template and can access values)
```

# Actions
```
{{ "Hello" }}
{{ quote "Hello" }}
{{ "Hello" | quote }}
{{ .Values.example }}
{{ toYaml .Values | indent 2 }}
```

# Types
https://helm.sh/docs/chart_template_guide/data_types/#helm
```yaml
text1: string
text2: "string"
int1: 1234
float1: 12.34
dict: {status: "upgraded"}
list:
  - 1
  - 2
  - 3
bool1: true
bool2: false
bool3: yes
bool4: no
bool5: on
bool6: off


```
# Functions
https://helm.sh/docs/chart_template_guide/function_list/#helm
```
{{ quote "hello" }}
{{ add 2 2 }}
{{ add 2 2 2 2 }}
{{ add 2 (add 2 2) }}
```

# Pipelines
https://helm.sh/docs/chart_template_guide/functions_and_pipelines/#pipelines
```
{{ "hello" | quote }}
{{ dict | toYaml | indent 4 }}
```

# Data
https://helm.sh/docs/chart_template_guide/builtin_objects/
```
{{ . }}
{{ .Values }}
{{ .Chart }}
{{ .Release }}
{{ .Subcharts }}
{{ .Files }}
{{ .Capabilities }}
```

# Strings 
https://helm.sh/docs/chart_template_guide/function_list/#string-functions
```
{{ print "1" "2" }} - concatenates

https://pkg.go.dev/fmt
{{ printf "key: %v value:%v" "a" "1" }}

{{ contains "a" "abc" }}
{{ "abc" | contains "a" }}
{{ hasPrefix "ab" "abc" }}
{{ hasSuffix "bc" "abc" }}

{{ trim " abc " }}
{{ " abc " | trim }}

asdf:
    {{ trim "key" }}: value
    
{{ indent 2 .Values.multilineText }}


{{ nindent 2 "asdf" }} indent and add new line at the begining when you put the action on the same line

{{ "3" }} -> number
{{ quote "3" }} -> string 

{{ substr 1 4 "01234567890" }} -> 123

{{ snakecase "appName" }} -> app_name
{{ camelcase "app-name" }} - appName
{{ kebabcase "appName" }} -> app-name

https://pkg.go.dev/regexp
{{ regexMatch "\d+" "1234 abc" }} -> 1234

{{ regexSplit "\d+" "a1b2c3" -1 }} -> [a,b,c]

# upper, lower, swapcase
# randAlpha, randAlphaNum, randAscii, randNumeric -> random text
# abbrev, abbrevboth -> text with ...
# initials -> Get first letter
# plural -> apple -> apples
# wrap, wrapWith
# title, untitle

```

# Numbers
Look at f - version 
```
{{ add 5 10 }} -> 5 + 10 = 15
{{ div 32 2 }} -> 32 / 2 = 16
{{ sub 10 5 }} -> 10 - 5 = 5
{{ mul 5.55 2 }}
{{ mulf 5.55 2 }} -> 

{{ ceil 9.71 }} -> 10
{{ floot 9.71 }} -> 9
{{ round 9.71 }} -> 10
{{ round 9.71 1 }} -> 9.7
{{ round 5.3 }} -> 5
{{ round 5.3 1 }} -> 5.3

{{ gt 10 5 }} 
{{ ge 10 5 }}
{{ lt 10 5 }}
{{ le 10 5 }}
{{ eq 5 5 }}
{{ ne 5 10 }}

{{ mod 10 5 }}
```

# Control flow
```
{{ if eq 2 2 }}
    output1
{{ else if eq 3 3 }}
    output2
{{ else }}
    output3
{{ end }}

{{ $data := .Values.someNumber }}
{{ $data | trim }}

{{ with .Values}}
    {{ .someNumber }}
    {{ $.Values.someNumber }}
{{ else }}
    No value bi4!
{{ end }}

# Remove whitespace from a code block
{{- with .Values -}}
    {{ .someNumber }}
{{- end -}}
```

# Lists
```
{{ list 1 2 3 4}}
{{ list (list 1 2 3 4) }}

{{ $list := list 1 2 3 4 }}

{{ range $list }}
     - {{ . }}
{{ end }}

{{ range $item := $list }}
     - {{ $item }}
{{ end }}

{{ range $index, $item := $list }}
     - {{ $index }}: {{ $item }}
{{ end }}

{{ index $list 0 }} -> 1
{{ index $list 1 }} -> 2
{{ index $list 2 }} -> 3
{{ len $list }}
{{ has $list 1}} -> true
{{ has $list 5}} -> false

{{ first $list }} -> 1 or nil if no elements
{{ uniq $list }}
{{ slice $list 0 2 }} -> 1, 2
{{ slice $list 1 }} -> 2, 3
{{ append $list 5 }} -> 1, 2, 3, 4, 5
{{ prepend $list 0 }} -> 0, 1, ..., 5
{{ concat $list $list }} -> 0 ... 5 0 ... 5
{{ without $list 1 }} -> 0, 2, 3. ... 5
{{ toJson $list }} -> [0, ... 5] 
{{ toYaml $list | indent N }} -> - 0,  ... - 5
{{ deepEqual $list1 $list2 }}
{{ sortAlpha $list  }} 


```

# Dicts
```
a: 1
b: 2

---
{{ dict
    "a" 1
    "b" 2
}}

{{ $dict :=
    dict "key" "value"
}}
outerObjects1: {{ $dict }}  # map[key:value}
outerObjects2: {{ toJSON $dict }}  # {"key": "value"}
outerObjects3: 
{{ toYaml $dict | indent 2 }}  # key: value

---

{{ .Values.Scores.key }}
{{ get .Values.Scores "key" }}
{{ get .Values.Scores "nonExistingsKey" }}  # returns ""
{{ hasKey .Values.Scores "key"}}
{{ $_ := set .Values.Scores "key" 30 }}
{{ set (deepCopy $dict) "key" 30 }}  # does not modify the original dict
{{ $_ := unset $dict "key" }}
{{ deepEqual $dict1 $dict2 }}
{{ $dect2 := omit $dict "key" }}

{{ keys $dict }}  # ["key"]
{{ range keys $dict }}
    - {{ . }}
{{ end }}

{{ values $dict }}  # ["value"] !unordered
{{ range values $dict }}  # unodreder
    - {{ . }}
{{ end }}

{{ split $dict }}

```

# Builtins
```
{{ kindOf $asdf }} # helm type
{{ typeOf $asdf }} # go type 
{{ typeIs "string" $asdf }}
{{ kindIs "string" $asdf }}

{{ toString $asdf}}
{{ toYaml $asdf }}
{{ toJson $asdf }}

{{ int $string }}
{{ int64 $string }}
{{ float64 $string }}
{{ float64 $int }}

{{ int "abc" }} # returns 0
{{ if regexMatch "[0-9]+" $string }}
    ...
{{ else }}
    Nah
{{ end }}

{{ fromYaml $string }}
{{ fromJson $string }} 
{{ fromYaml (.Files.Get "other-values.yaml") }}

```

# Named templates
```
{{- define "mytemplate" }}
    Hello my template!
{{- end }}

{{/* My comment */}}

{{- include "mytemplate" . }} # returns the value so we can pipe to indent for example
{{- template "mytemplate" . }} # renders directly
```
You can put templates *.tpl files in templates/ directory.

# Dependencies (aka. sub-charts)

```
dependencies: 
    - name: values-checker
      version: "0.1.0"
      repository: "file://../values-checker"
```
`helm dep list`
`helm dep build` # fetches the missing dependencies
`helm dep upgrade <chart>` # updates a chart

Overriding values in the parent chart
```
data:
    values-checked: # put the name of the chart
        exampleKey: exampleValue
```

Global values (access like .Values.global.globalKey)
```
global:
    globalKey: globalValue
```

# Tests
```
suite: My suite
templates:
  - my-template.yaml
tests:
  - it: |
      Should test ...
    set:
      TestDictionary:
        key: value2
        newProperty: newValue
    asserts:
      - equal:
          path: data.echoedDictionary
          value: 
            key : 2
```

# Date and time

{{ duration 120 }} # 2m
{{ durationRound "2500h5m3s" }} # 3m
{{ ago (dateModify "-20m" now) }} # 20m
{{ unixEpoch now }}
{{ htmlDate now }}
{{ dateInZone $format now "UTC" }} 
{{ htmlDateInZone $format now "UTC" }} 

# Helm repos
```
helm repo add ....
helm install <release> <chart>
helm repo list
helm repo remove <repo-name>
helm search hub <string>
helm search repo <string>
helm pull <repo name>/<chart name>
helm pull <repo name>/<chart name> --untar
helm package <directory>/
helm repo index <directory>/
helm repo index <directory>/ --url https://example.com
```

# CRDs
lives in crds/ directory
`helm install crds crds-example`

```
kubectl get crd
helm install test-crds crds-example
kubectl get crd
kubectl get students
helm uninstall test-crds
kubectl delete crd/students.example.com
```

# Overrides
`helm [command] --set input="Tralala"`
`helm [command] --values values2.yaml`

# Chart
```yaml
apiVersion: v2
name: my-chart
description: Bla bla
type: application
version: 1.0.0 # chart version
appVersion: 0.0.1 # your app version
```

# .helmignore
* Like docker ignore you can exclude files
* Excludes sensitive information like secrets
* Excludes build files ...

# Helm release lifecycle
```bash
Not deployed -> helm install -> Pending install -> succes -> Deployed
Not deployed -> helm install -> Pending install -> failure -> Failed
Failed -> helm upgrade -> Pending install or upgrade -> success -> Deployed
Deployed -> helm upgrade -> Pending upgrade -> success - Deployed
Deployed -> helm rollback -> Pending upgrade -> success -> Deployed
Deployed -> helm uninstall -> Not deplyed
```

# Helm template

Sprig gives additional functions.
https://masterminds.github.io/sprig/

```bash
helm template .
helm template values-checker
helm template values-checker --set exampleKey=asdf
helm template values-checker --set name=asdf
helm template values-checker --values values-checker/values-override.yaml
helm template values-checker -f values-checker/values-override.yaml
```

# Helm install
```bash
helm install <release name> <chart>
helm install first-release values-checker/
helm list
helm status first-release
kubectl get configmap/helm-values -o yaml
```

# Helm upgrade
```bash
helm upgrade first-release values-checker/
helm upgrade first-release values-checker/ --set newState=upgraded 
helm get all first-release
helm get manifest first-release
helm upgrade --install first-release values-checker/
kubectl get secrets
kubectl get secrets sh.helm.release.v1.first-release.v8 -o yaml
helm history first-release
```

# Helm uninstall
```bash
helm uninstall first-release
```

# Helm rollback
```bash
helm rollback first-release 
```

# Helm create
```bash
helm create <chart name>
helm create hello-world
cd hello-world/
```

