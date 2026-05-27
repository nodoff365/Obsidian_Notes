1. IaC(Infrastructure As Code)
1.1 자동화
1.2 멱등성
1.3 휴먼에러방지
1.4 경제적이득증가

2. IaC 도구 종류
 2.1 구성 관리
  2.1.1 Puppet
  2.1.2 Ceaf
  2.1.3 Saultstack
  2.1.4 Ansible
  
 2.2 배포 관리
  2.2.1 Terraform: 다양한 플랫폼에서 사용 가능
  2.2.2
  
https://learn.microsoft.com/ko-kr/cli/azure/install-azure-cli-windows?view=azure-cli-latest&pivots=msi
https://developer.hashicorp.com/terraform/install
http://registry.terraform.io/
https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_network
https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs


```bash
terraform


terraform plan
terraform apply --auto-approve
terraform destroy --auto-approve
```

```bash
az login

```