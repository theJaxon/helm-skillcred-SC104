# helm-skillcred-SC104
Preparing for [Developing Helm Charts (SC104)](https://training.linuxfoundation.org/skillcred/helm/) - a credential by LinuxFoundation

### Basic commands
#### Getting release info
```bash
helm get all

helm get manifest

helm get notes

helm get values

helm get hooks
```

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
