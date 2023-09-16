# Installing Helm

#### https://helm.sh/docs/intro/install/

<img src='https://www.ozone.one/wp-content/uploads/2023/05/HELM-1024x536.png'/>
Image source: Ozone: https://www.ozone.one/what-is-helm-a-comprehensive-overview-for-beginners/

### From Script
<br>
Helm now has an installer script that will automatically grab the latest version of Helm and install it locally.
<br>
You can fetch that script, and then execute it locally. It's well documented so that you can read through it and understand what it is doing before you run it.

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version

```
