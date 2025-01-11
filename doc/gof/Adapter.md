# Terraform 适配器模式 (Adapter Pattern)

## 概述
适配器模式使得不兼容的接口可以协同工作。在Terraform中，这种模式特别适用于统一管理不同云服务商的资源，使得上层应用可以通过统一的接口来操作不同的云服务。

## 实现示例
以下示例展示了如何创建一个统一的存储模块来适配不同云服务商的存储服务：

### 模块结构
```
modules/
  └── storage/
      ├── variables.tf
      ├── main.tf
      └── outputs.tf
```

### 变量定义
```hcl:modules/storage/variables.tf
variable "provider_type" {
  description = "Cloud provider type (aws or gcp)"
  type        = string
}

variable "bucket_name" {
  description = "Name of the storage bucket"
  type        = string
}

variable "location" {
  description = "Location/region for the bucket"
  type        = string
}
```

### 主要实现
```hcl:modules/storage/main.tf
resource "aws_s3_bucket" "aws_storage" {
  count  = var.provider_type == "aws" ? 1 : 0
  bucket = var.bucket_name
  
  tags = {
    Environment = "Production"
    Provider    = "AWS"
  }
}

resource "google_storage_bucket" "gcp_storage" {
  count    = var.provider_type == "gcp" ? 1 : 0
  name     = var.bucket_name
  location = var.location

  labels = {
    environment = "production"
    provider    = "gcp"
  }
}

# 统一的输出接口
output "bucket_id" {
  value = var.provider_type == "aws" ? (
    length(aws_s3_bucket.aws_storage) > 0 ? aws_s3_bucket.aws_storage[0].id : ""
  ) : (
    length(google_storage_bucket.gcp_storage) > 0 ? google_storage_bucket.gcp_storage[0].name : ""
  )
}
```

### 使用示例
```hcl:main.tf
module "aws_storage" {
  source        = "./modules/storage"
  provider_type = "aws"
  bucket_name   = "my-aws-bucket"
  location      = "us-west-2"
}

module "gcp_storage" {
  source        = "./modules/storage"
  provider_type = "gcp"
  bucket_name   = "my-gcp-bucket"
  location      = "US-WEST1"
}
```

## 设计要点
1. **统一接口**: 通过模块提供统一的输入和输出接口
2. **隐藏实现**: 屏蔽不同云服务商的实现细节
3. **条件创建**: 使用 count 参数根据 provider_type 选择性创建资源
4. **统一输出**: 提供统一的输出接口，方便上层应用使用

## 优势
1. 简化了多云环境的资源管理
2. 提高代码复用性
3. 降低维护成本
4. 便于扩展新的云服务商支持
5. 统一的接口减少了学习成本

## 使用场景
- 多云环境资源管理
- 统一不同服务的配置方式
- 需要在不同云服务商之间切换时
- 封装第三方服务接口

## 注意事项
1. 需要考虑不同云服务商的特性差异
2. 接口设计要足够抽象以适应不同实现
3. 注意处理不同服务商的错误情况
4. 考虑资源的生命周期管理 