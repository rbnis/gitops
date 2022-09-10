# GitOps Repo

## Setup

### Set some variables

```bash
export CLUSTER_NAME="<your-cluster-name>"
export GITHUB_TOKEN="<your-token>"
export GITHUB_USER="<your-username>"
```

### Encryption

Here we will create two Age Private and Public keys. Using [SOPS](https://github.com/mozilla/sops) with [Age](https://github.com/FiloSottile/age) allows us to encrypt secrets and use them in Flux and work convieniently with them.

1. Create Age Keypairs.

```bash
mkdir -p "$HOME/.config/sops/age"
age-keygen -o "$HOME/.config/sops/age/keys.txt"
age-keygen -o "$HOME/.config/sops/age/cluster.keys.txt"
```

2. Create `.sops.yaml` file.

```bash
export AGE_CLUSTER_PUBLIC_KEY="$(cat $HOME/.config/sops/age/cluster.keys.txt |  awk '(NR==2)' | sed 's/.*: //')"
export AGE_PERSONAL_PUBLIC_KEY="$(cat $HOME/.config/sops/age/keys.txt |  awk '(NR==2)' | sed 's/.*: //')"

envsubst < ./.template/.sops.yaml > ./.sops.yaml
```

3. Export the `SOPS_AGE_KEY_FILE` variable in your `bashrc`, `zshrc` or `config.fish` and source it.

```bash
echo 'export SOPS_AGE_KEY_FILE="$HOME/.config/sops/age/keys.txt"' >> "$HOME/.zshrc"
source "$HOME/.zshrc"
```

4. Create a secret in the cluster to decrypt secrets.

```bash
kubectl create namespace flux-system
cat "$HOME/.config/sops/age/cluster.keys.txt" | kubectl create secret generic sops-age --namespace=flux-system --from-file=age.agekey=/dev/stdin
```

### FluxCD

Bootstrap flux with:
```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=<your-repo> \
  --branch=main \
  --path=clusters/$CLUSTER_NAME \
  --personal
```

### Pre-Commit

```sh
pre-commit install-hooks
pre-commit install
```
