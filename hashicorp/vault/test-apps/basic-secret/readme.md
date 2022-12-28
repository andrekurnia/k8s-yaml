# How to integrate vault in k8s and the deployment

In order for us to start using secrets in vault, we need to setup a policy.

```
## Create a role for our app
# Access vault pod via SSH
kubectl -n <your-vault-namespace> exec -it <your-vault-pod> sh 

# Create new authentications for your serviceAccount in the cluster into the Vault
vault write auth/kubernetes/role/basic-secret-role \
   bound_service_account_names=basic-secret \
   bound_service_account_namespaces=<your-serviceaccount-namespace> \
   policies=basic-secret-policy \
   ttl=1h
```

The above maps our Kubernetes service account, used by our pod, to a policy.
Now lets create the policy to map our service account to a bunch of secrets


```
## We want the service account only can read secrets that stored with the exact path stated.
# Access vault pod via SSH
kubectl -n <your-vault-namespace> exec -it <your-vault-pod> sh

# You can create create the command starts from "path..." into a file using vi/vim.
cat <<EOF > /home/vault/app-policy.hcl
path "secret/basic-secret/*" {
  capabilities = ["read"]
}
EOF

# Apply the policy config into vault system.
vault policy write basic-secret-policy /home/vault/app-policy.hcl

```

Now our service account for our pod can access all secrets under `secret/basic-secret/*`
Lets create some secrets.


```
# Access vault pod via SSH.
kubectl -n <your-vault-namespace> exec -it <your-vault-pod> sh 

# Enable kv that used to store your secrets. Path can be customized as what you wish.
vault secrets enable -path=secret/ kv

# Create the secrets and it will be automatically stored in the chosen path (customized as well).
vault kv put secret/basic-secret/helloworld username=dbuser password=sUp3rS3cUr3P@ssw0rd
```

Lets deploy our app and see if it works!

```
# Apply the yaml file
kubectl -n <your-vault-namespace> apply -f deployment.yaml

# See the status of your pod
kubectl -n <your-vault-namespace> get pods
```
