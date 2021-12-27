# system-engineer-test

### Virtual Machine Template

```
main.tf

terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "3.20.0"
    }
  }
}

provider "google" {
  credentials = file("cobalt-howl.json")

  project = "cobalt-howl-336209"
  region  = "asia-southeast2"
  zone    = "asia-southeast2-a"
}

resource "google_compute_network" "nginx_network" {
  name = "nginx-vpc-1"
  auto_create_subnetworks = "false"
}

resource "google_compute_subnetwork" "nginx_subnet1" {
  name = "nginx-vpc-1"
  ip_cidr_range = "10.1.0.0/24"
  region = "asia-southeast2"
  network = google_compute_network.nginx_network.name
}

resource "google_compute_firewall" "rules" {
  name = "nginx-rules"
  network = "nginx-vpc-1"
  allow {
    protocol = "tcp"
    ports    = ["80","443","22"]
  }
  allow {
    protocol = "icmp"
  }  
}

resource "google_compute_instance_template" "tpl" {
  name = "nginx-instance-template1"
  machine_type = "e2-small"

  disk {
    source_image = "ubuntu-os-cloud/ubuntu-2004-lts"
    auto_delete  = true
    disk_size_gb = 10
    boot         = true
  }

  network_interface {
    network = google_compute_network.nginx_network.name
    subnetwork = google_compute_subnetwork.nginx_subnet1.name
    access_config {
      //ephemeral ip address 
    } 
  }
}


resource "google_compute_instance_from_template" "tpl" {
  name = "nginx-server"
  zone = "asia-southeast2-a"
  
  source_instance_template = google_compute_instance_template.tpl.id
  metadata_startup_script = file("install-docker-nginx.sh")
}

```

```
install-docker-nginx.sh
#! /bin/bash
apt update -y
apt upgrade -y
apt install nginx -y
service nginx start
snap install docker --classic
service docker start
```


