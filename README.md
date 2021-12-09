# Terraform Notes

- Wait untill a resource is available.

``` bash
 resource "null_resource" "check_resource" {
   provisioner "local-exec" {
     command = "while [ \"$result\" != \"Success\" ]; do result=$(aws ses get-identity-verification-attributes --identities  ${var.domain} --region us-east-1 --query VerificationAttributes.\"*\".VerificationStatus --output text);echo $result; done"
   }
 }
```

- Print a variable to a file

``` bash
resource "null_resource" "terraform-debug" {
  provisioner "local-exec" {
    command = "echo $VARIABLE1 >> debug.txt ; echo $VARIABLE2 >> debug.txt"

    environment = {
        VARIABLE1 = jsonencode(var.your_variable_name)
        VARIABLE2 = jsonencode(local.piece_of_data)
    }
  }
}
```

- ECR find latest tagged image.

``` bash
data "external" "latest_image" {
  program = [
    "aws", "ecr", "describe-images",
    "--repository-name", "REPO_NAME",
    "--query", "{\"tags\": to_string(sort_by(imageDetails,& imagePushedAt)[-1].imageTags)}",
  ]
}

data "aws_ecr_repository" "repo" {
  name = "REPO_NAME"
}

locals {
container_image       = join(":", [data.aws_ecr_repository.repo.repository_url, trim(jsonencode(data.external.latest_image.result.tags), "[]\"\\")])
}
```

- Sleep 30 seconds

``` bash
resource "time_sleep" "wait_30_seconds" {
  depends_on = [previous_resource.resource]

  create_duration = "30s"
}

resource "next_resource" "resource" {
depends_on = [
    time_sleep.wait_30_seconds,
    ]
}
```
