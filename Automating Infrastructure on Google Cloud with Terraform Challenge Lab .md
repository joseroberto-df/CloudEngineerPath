## `Nome do Laboratório` - *Automating Infrastructure on Google Cloud with Terraform: Challenge Lab [GSP345]*

## Tarefa 1. Crie os arquivos de configuração

Execute os comandos abaixo no terminal Cloud Shell

touch main.tf
touch variables.tf
mkdir modules
cd modules
mkdir instances
cd instances
touch instances.tf
touch outputs.tf
touch variables.tf
cd ..
mkdir storage
cd storage
touch storage.tf
touch outputs.tf
touch variables.tf
cd


* (Cole-o em variável.tf )

variable "region" {
 default = "YOUR_REGION"
}

variable "zone" {
 default = "YOUR_ZONE"
}

variable "project_id" {
 default = "PROJECT_ID"
}

* (Cole-o em main.tf )

terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}

module "instances" {
  source     = "./modules/instances"
}



Execute isso no CloudShell:
 
terraform init 

#############################################################################

## Tarefa 2. Importar infraestrutura

* (Cole-o em module/instances/instances.tf )

resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-1"
  zone         = "ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-1"
  zone         =  "ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}


Execute isso no CloudShell

terraform import module.instances.google_compute_instance.tf-instance-1 [INSTANCE_ID_1]

terraform import module.instances.google_compute_instance.tf-instance-2 [INSTANCE_ID_2]

terraform plan
terraform apply


#############################################################################

## Tarefa 3. Configurar um back-end remoto

* Cole o código abaixo em storage/storage.tf


resource "google_storage_bucket" "storage-bucket" {
  name          = "YOUR_BUCKET_NAME"
  location      = "us"
  force_destroy = true
  uniform_bucket_level_access = true
}


* Adicione as seguintes linhas ao main.tf

d
module "storage" {
  source     = "./modules/storage"
}


Execute isso no CloudShell

terraform init
terraform apply


* Atualize o seguinte código para main.tf

terraform {
  backend "gcs" {
    bucket  = "YOUR_BUCKET_NAME"
 prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

* Execute isso no CloudShell

terraform init

###########################################################################

## Tarefa 4. Modificar e atualizar a infraestrutura

* Adicione o seguinte código no instance.tf

resource "google_compute_instance" "INSTANCE_NAME" {
  name         = "YOUR_INSTANCE_NAME"
  machine_type = "n1-standard-2"
  zone         = "ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

* Execute isso no CloudShell

terraform init
terraform apply

#############################################################################

## Tarefa 5. Destruir recursos

terraform taint module.instances.google_compute_instance.INSTANCE_NAME

terraform init
terraform apply


* Vá e `remova instance-3` de `instance.tf`

terraform apply

#############################################################################

## Tarefa 6. Use um módulo do Registro


* (Cole-o seguinte código em main.tf)


module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "PROJECT_ID"
    network_name = "VPC_NAME"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "us-east1"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "us-east1"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "Subscribe TO ATUL GUPTA"
        },
    ]
}


* Execute isso no CloudShell

terraform init
terraform apply

*Vá para Instance.tf e atualize todos com o seguinte código

resource "google_compute_instance" "tf-instance-1"{
  name         = "tf-instance-1"
  machine_type = "n1-standard-2"
  zone         ="ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "VPC_NAME"
     subnetwork = "subnet-01"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2"{
  name         = "tf-instance-2"
  machine_type = "n1-standard-2"
  zone         = "ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "VPC_NAME"
     subnetwork = "subnet-02"
  }

  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}


* Execute isso no CloudShell

terraform init
terraform apply

#############################################################################

## Tarefa 7. Configurar um firewall

* Adicione o seguinte código em main.tf

resource "google_compute_firewall" "tf-firewall"{
  name    = "tf-firewall"
 network = "projects/YOUR_PROJECT_ID/global/networks/YOUR_VPC_Name"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}


* Execute isso no CloudShell

terraform init
terraform apply


# Parabéns! Você concluiu este laboratório de desafio.