# Akeyless Kubernetes Authentication External Gateway Example

## Prerequisites

- Make sure that you have your kubectl configured to connect to the correct target remote cluster
- An Akeyless Admin to your account may be required to grant access to create kubernetes auth configs
- The gateway with network access to the kubernetes cluster MUST have TLS configured
- Installation of External Secrets Operator is out of scope for this example

## Configure Kubernetes Auth Access Permissions

### Only Gateway **Admin** users can access and manage the Access Permissions settings.

To configure **Access Permissions** in your Gateway Configuration Manager, follow these steps:

1. Navigate to the **Access Permissions** tab.

2. Click **New** to create a new access permission item.

3. Provide a meaningful **Name** for the item, such as "Kubernetes Auth Managers" or "Dynamic Secrets Admin".

4. From the **Auth Method** drop-down menu, select the relevant Authentication Method and set the exact Sub-Claims identifying your users. Click **Next** to proceed.

5. In the **Permission Settings** section, choose either **Admin** or **Custom**.

6. If you select **Custom**, you can specify the relevant permissions to grant to the selected Auth Method. For example, you can grant permissions for managing **Kubernetes Auth Managers** or other specific operations. Select the desired permissions and click **Finish**.

Based on the selected operations, the relevant Auth Method will have access to initiate only those specific operations.

Remember, only Gateway **Admin** users have the ability to access and manage the Access Permissions settings.

By configuring Access Permissions, you can control and limit the operations that different Auth Methods can perform within the Gateway, ensuring a secure and granular access control system.

## Create the Kubernetes Auth Method

### Create the kubernetes service account and cluster role binding of delegator access

> The `k8s_auth.yaml` can be found below

```sh
kubectl create -f k8s_auth.yaml
```

Extract the bearer token from the kubernetes service account secret we just created.

```sh
kubectl get secret gateway-token-reviewer-token -n akeyless-auth -o jsonpath='{.data.token}' | base64 --decode
```

### Create the Kubernetes Auth Method and K8s Auth Config

Display the kubernetes cluster host endpoint and CA certificate (mac/linux)

```bash
kubectl config view --flatten --minify --output=jsonpath='{.clusters[0].cluster.server}{"\n"}{.clusters[0].cluster.certificate-authority-data}' | awk 'NR==1{print "Host: "$0} NR==2{print "CA Certificate: "; system("echo "$0" | base64 --decode")}'
```

Display the kubernetes cluster host endpoint and CA certificate (Windows Powershell)

```powershell
kubectl config view --flatten --minify --output=jsonpath="{.clusters[0].cluster.server}{"`n"}{.clusters[0].cluster.certificate-authority-data}" | ForEach-Object {
    if ($_.Trim() -match '^[A-Za-z0-9+/=]+$') {
        "CA Certificate: "
        [System.Convert]::FromBase64String($_) | ForEach-Object {[System.Text.Encoding]::UTF8.GetString($_)}
    } else {
        "Host: $_"
    }
}
```

Navigate to [console.akeyless.io/auth-methods](https://console.akeyless.io/auth-methods) and create a new Kubernetes Auth Method associated to the gateway with network access to the target kubernetes cluster's host endpoint.

The auth method and the k8s auth config will have been created automatically.

Navigate to the k8s auth config screen of gateway we just added the k8s auth config to and note the FULL name of the k8s auth config (for example like https://gw-config.example.com/kubernetes)

## Create and configure the "k8s" namespace

This will create the "k8s" namespace and the service account "cg-akl-azure-k8s-sa"

```sh
kubectl apply -f k8s-secrets.yaml
```

## Create the ESO Secret Store

```sh
kubectl apply -f eso-secret-store.yaml
```

## Create the ESO External Secret

```sh
kubectl apply -f eso-external-secret.yaml
```

We note there is now an error stating the access ID for the kubernetes auth method is not authorized to access the secret.

We need to create an Access Role and associate the kubernetes auth method to it and then we need to add an item rule allowing Read and List access to the target secret.



