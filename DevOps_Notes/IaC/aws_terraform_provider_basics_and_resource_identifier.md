# AWS Terraform Provider 学习笔记

## 基础介绍

-   **AWS Terraform provider** 允许你在 AWS 中管理资源。

-   通过在 Terraform 配置文件中添加资源块来创建资源。

-   资源块的基本语法如下：

    ``` hcl
    resource "资源类型" "标识符" {
      参数1 = 值1
      参数2 = 值2
    }
    ```

## 关于资源类型

-   资源类型的命名以提供商名称开头，AWS 的所有资源类型都以 `aws_` 开头。
-   完整的 AWS 资源类型列表可在 AWS provider 文档页面的侧边栏中找到。

## 实践示例：创建 Amazon VPC

-   配置一个 Amazon VPC（虚拟私有云）资源的配置如下：

    ``` hcl
    resource "aws_vpc" "web_vpc" {
      cidr_block = "192.168.100.0/24"
      enable_dns_hostnames = true
      tags = {
        Name = "Web VPC"
      }
    }
    ```

-   **配置说明**：

    -   `cidr_block`：VPC 的 IP 地址范围（必需参数）。
    -   `enable_dns_hostnames`：启用 DNS 主机名（可选参数）。
    -   `tags`：资源标签，这里设置了名称为 "Web VPC"（可选参数）。

## 操作说明

-   使用 `cat` 命令将这个 VPC 配置追加到现有的 `main.tf` 文件中。
-   完整的 `aws_vpc` 资源参数列表可以在资源文档页面中找到。

## 标识符的作用

-   **代码引用**：在 Terraform 配置的其他部分引用这个资源时使用，例如：

    ``` hcl
    resource "aws_vpc" "web_vpc" {
      cidr_block = "192.168.100.0/24"
    }

    resource "aws_subnet" "web_subnet" {
      vpc_id = aws_vpc.web_vpc.id  # 这里引用了上面VPC的id属性
    }
    ```

-   **本地识别**：仅在 Terraform 配置文件中使用，不会显示在 AWS 控制台。

-   **状态管理**：Terraform 使用这个标识符在状态文件中跟踪资源。

## 标识符与 Name 标签的区别

-   **标识符**（例如 `web_vpc`）：
    -   仅在 Terraform 配置文件中使用。
    -   只对 Terraform 有意义，用于代码内部引用。
    -   更改标识符会导致资源重新创建。
-   **tags 中的 Name**（例如 `Name = "Web VPC"`）：
    -   实际显示在 AWS 控制台中。
    -   用于在 AWS 层面识别和管理资源。
    -   可以随时修改而不会影响资源本身。

## 实际使用建议

-   **标识符**：使用简短、符合编程规范的名称，便于代码引用。
-   **Name 标签**：使用更详细、人类可读的描述，便于在 AWS 控制台中识别。
-   两者可以不同，但保持相关性有助于代码维护。

## 最佳实践示例

``` hcl
resource "aws_vpc" "prod_web_vpc" {  # Terraform内部使用的标识符
  cidr_block = "192.168.100.0/24"
  tags = {
    Name = "Production Web Application VPC"  # AWS控制台显示的名称
    Environment = "Production"
    Project = "WebApp"
  }
}
```
