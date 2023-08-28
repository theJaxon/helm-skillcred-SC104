# helm-skillcred-SC104
Preparing for [Developing Helm Charts (SC104)](https://training.linuxfoundation.org/skillcred/helm/) - a credential by LinuxFoundation

### Basic commands
#### Getting release info
```bash
helm get all

helm get manifest

helm get notes

helm get values

# Get all values used by a release (Including defaults)
helm get values --all

helm get hooks
```

#### Getting chart related info
```bash
helm show all # Show all chart information

helm show chart # Display Chart definition

helm show readme 

helm show values
```

---

### [Create charts using basic templating](https://helm.sh/docs/helm/helm_create/)

```bash
helm create <chart-name>
```

#### Chart dependencies 
In the dependencies map in `Chart.yaml` many fields are supported
- The mandatory ones are **name, repository and version**
- `condition`: Boolean value that determines whether the dependency should be included or not 
- `Tags`: A list of Boolean values that determine whether the chart should be included or not
- `import-values`: Mapping of source values to parent values
- `Alias`: alternative name given to the dependency (Used when working with multiple versions of same dependency)

```yaml
# Conditionally install redis dependency
dependencies:
  - name: redis
    repository: https://charts.bitnami.com/bitnami
    version: 18.0.0
    condition: redis.enabled

# By default if not specified in values, the condition will be ignored meaning dependency will be installed anyway unless explicitly specified as false
helm install . --generate-name # redis dependency is included 

# Here redis dependency won't be installed
helm upgrade <release-name> <chart-location> --set redis.enabled=false
```

```yaml
dependencies:
  - name: mariadb
    repository: https://charts.bitnami.com/bitnami
    version: 13.1.1
    tags:
      - backend
      - database
  - name: redis
    repository: https://charts.bitnami.com/bitnami
    version: 18.0.0
    tags:
      - backend
      - cache

# Both dependencies will be installed
helm upgrade --install jaxon .

# backend tag is disabled so that all dependencies aren't installed
# Then only redis is picked for installation by enabling the cache tag
helm upgrade --install jaxon . --set tags.backend=false --set tags.cache=true
```
- Both conditions and tags can be combined, however conditions **always overrides** tags

#### Basic Templating
##### Objects
###### 1. Release object
- Used to get `helm release` metadata
- `.Release.Name` and `.Release.Namespace` are commonly used
- `.Release.IsUpgrade` and `.Release.IsInstall` The first boolean is for whehter there's an upgrade or rollback, the second one is for installation

##### Control Structure
###### If/Else
- Conditional blocks for resource creation
```yaml
{{- if .Values.configmap.enabled  }}
data:
{{- $configmap := printf "configs/%s/%s/config.yaml" .Values.name .Release.Namespace }}
{{ (.Files.Glob $configmap ).AsConfig | indent 2 }}
{{- end }}
```

###### with
- Modifies the scope of the resources being generated

---

### Testing helm charts
#### Templating and validating
- Charts can be tested with client-side rendering via `helm template` command
- In order to have server side validation one can add **--validate** to the above command or can try running `helm install --dry-run`

#### Testing on a live kubernetes cluster
Can be broken into 2 ways
1. Readiness probes with `helm install --wait` which makes return success status only once readiness probe passes
2. Test hooks and `helm test` command (Test hooks are just pods that do some specific task after `helm install` is executed)
  - You specify that it's a test hook via **helm.sh/hook: test** annotation
