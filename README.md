# googlecloud-cheatsheet

## Terraform (Basic Example)

#### Install  
wget URL
unzip FILENAME
cd $HOME
source ~/.bashrc

#### Launch Code Editor from Cloud Shell
Launch Cloud Shell<br>
Open code editor

#### Create Terraform files
mkdir DIRECTORY_NAME<br>
Open code editor<br>
Create following files under DIRECTORY_NAME:<br>

- provider.tf
- management.tf
- privatenet.tf
- othernet.tf
- ./instance/main.tf

<details>
<summary>template</summary>
  
```
Details go here
```
</details>

<details>
<summary>provider.tf</summary>
  
```
provider "google" {}
terraform init
```
</details>


