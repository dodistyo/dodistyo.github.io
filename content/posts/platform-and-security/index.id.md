---
weight: 10
title: "Cerita Tentang Platform Engineering dan Keamanan Siber"
date: 2025-06-20T19:37:00+07:00
lastmod: 2025-06-20T19:37:00+07:00
draft: false
author: "Dodi"
authorLink: "https://github.com/dodistyo"
description: "Cerita gue"
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"
tags: ["security", "platform", "devops"]
categories: ["security", "platform"]
hiddenFromHomePage: false
comment:
  enable: true
---

Oke, ini cerita dari orang yang udah nyemplung banget di dunia platform dan security. Ngebangun sesuatu, ngehancurin, ngamanin lagi, terus ngebenerin yang rusak. Ya, begitulah biasanya.  
<!--more-->

## Menghubungkan dan Mengamankan Infrastruktur Hybrid {#connect-and-secure-infra}

Sekarang dunia IT tuh berputar di infrastruktur hybrid — campuran antara data center di kantor, beberapa lokasi on-prem, dan workload di cloud (khususnya GCP). Tantangan pertama yang harus diselesaikan adalah mengamankan lalu lintas antar *environtment* itu.  

Buat nyambungin dan ngamanin traffic **north-south** (antar *environment*), kita pakai firewallnya `FortiGate` dan fitur SD-WAN-nya. Solusi ini bisa nyatuin semua site, termasuk cloud, jadi satu jaringan privat dan aman.  

Terus buat aplikasi yang menghadap ke publik, kita pilih `FortiWeb` sebagai WAF-nya. Alasannya? Karena kita butuh solusi WAF yang bisa ngamanin aplikasi cloud sekaligus yang ada di on-prem juga. Kita pasang ini di GCP pakai lisensi PAYG biar fleksibel buat scaling dan budgeting. Supaya makin andal, kita deploy dalam mode High Availability (HA) dengan instance Active-Active. Memang biayanya jadi dobel, tapi ya itu harga yang harus dibayar buat sistem yang bisa diandalkan.  

Setelah itu, kita fokus ke pengamanan traffic **dalam** Kubernetes cluster (traffic east-west). Strateginya ada dua: enkripsi dan kontrol akses.

1. **Enkripsi**: Untuk bisa pakai pendekatan Zero Trust, semua komunikasi antar service harus terenkripsi. Kita pakai `Linkerd` karena ringan, dan bisa otomatis kasih enkripsi end-to-end pakai mutual TLS (mTLS).  

2. **Kontrol Akses**: Meski Linkerd udah enkripsi, kita masih perlu ngatur service mana yang boleh ngobrol sama siapa. Di sini kita pakai `Cilium` sebagai container network interface. Dia bisa ngatur network policy sampai ke level L4 dan L7, jadi kita bisa kontrol traffic dengan lebih detail.  

Biar makin aman dan operasionalnya otomatis, kita bikin service `security and operation` sendiri. Ini kayak API pusat yang ringan buat nanganin tugas-tugas operasional umum. Salah satu fiturnya terhubung dengan *tools* keamanan runtime kita, `Falco`. Jadi kalau Falco nemu aktivitas mencurigakan, service ini langsung bikin network policy buat ngisolasi pod yang kena. (Walaupun nggak lama setelah itu, muncul `Falco Talon` yang bisa otomatis nanganin hal ini juga.)  

Setelah infrastrukturnya beres, tantangan berikutnya muncul di layer aplikasi — terutama soal manajemen rahasia (secrets). Ada beberapa opsi: bisa pakai secret manager-nya GCP, atau kalau udah full GitOps bisa pakai sealed secrets atau sops. Tapi kita pilih `HashiCorp Vault` karena bisa jadi pusat manajemen credential yang aman dan gampang diatur.

## Membangun Ekosistem Platform {#build-platform}

Kita sepenuhnya pakai prinsip Infrastructure as Code (IaC) buat ngatur semuanya. Semua resource cloud kita ubah jadi kode. Kita pilih `Terraform`, karena hampir semua dokumentasi GCP ada kode terraformnya. (Walau sekarang Terraform udah nggak open-source dan kita bisa aja pindah ke `OpenTofu`.)

Biar workload di Kubernetes jalan dengan efisien, kita juga harus ngatur node-nya dengan baik. Scaling node pool manual itu nggak efisien. Jadi, kita manfaatin fitur `Node Auto-Provisioning (NAP)` dari GKE. Ini kayak `Karpenter` di AWS, tapi built-in di GKE. NAP bisa nambah atau ngurangin node pool dengan jenis mesin yang sesuai sama kebutuhan real-time pod kita. Jadi resource tetap optimal dan hemat biaya.

Sebelum aplikasi masuk ke proses GitOps, kita lewatin dulu pipeline CI/CD. Tantangannya, kita butuh pipeline build yang efisien tanpa harus maintain banyak agen statis. Kita pakai `Jenkins`, tapi dengan pendekatan modern. Agen Jenkins-nya diprovide secara dinamis lewat Jenkins Kubernetes Operator, yang bikin agen jalan sebagai pod dalam cluster.  

Supaya hemat banget, pod agen ini jalan di GCP Spot VMs — auto-scale ke nol kalau nggak ada pipeline yang jalan. Ini bikin biaya build turun sampai 94% dibanding pakai runner yang selalu aktif. Iya, Spot VMs emang kece badai.  

Lanjut ke proses deployment aplikasi. Nge-deploy aplikasi ke banyak cluster Kubernetes, apalagi di lingkungan hybrid, itu ribet. Kita pakai `ArgoCD` buat manage semuanya. Solusi ini bisa handle GitOps deployment dari satu titik ke semua cluster. Dibanding `FluxCD`, ArgoCD punya visibilitas terpusat dan ga nambah sistem *footprint* di sisi cluster target.

Meskipun job title-nya bukan developer, gue masih ngoding terus. Kaya misalnya custom service yang gue bikin kemarin itu pakai `Rust`. Emang bukan bahasa yang paling produktif sih menurut gue, tapi cara Rust nge-handle memory ngajarin kita buat nulis kode lebih rapi dan aman. Di Rust ga ada garbage collector makannya bikin performa lebih bagus.

Biar makin ringan dan lebih oke, kita *build* servicenya pakai base image `scratch`. Jadi kontainernya benar-benar minimum, aman, dan kecil. Ini juga ngebantu ngurangin *attack surface* dari container kita — nilai plus buat hardening keamanan.

## Tetap Relevan {#stay-relevant}

Biar makin paham gimana sistem terdistribusi bekerja, gue ngulik algoritma konsensus `RAFT` yang dipakai `etcd` (komponen penting di Kubernetes). Bahkan gue sempet bikin ulang versi Redis yang terdistribusi dengan teknik `consistent hashing`. Ya, kadang "reinvent the wheel" itu bukan ide buruk.

Dan ya, sekarang siapa sih yang gak ngomongin AI. Salah satu proyek terbaru gue adalah otomatisasi code review. Kita bikin workflow automation sederhana pakai `n8n`, dan `Gemini LLM` jadi otaknya buat analisa kode dan flagging issue. Meskipun n8n bukan termasuk *Agentic AI* yang sejati, tapi buat kasus kita udah cukup. Bahkan kita berani auto-merge perubahan kecil di MR.

Tahun ini benar-benar rewarding. Dari ngamanin perimeter sampai ke tiap workload, dan otomatisasi dari IaC sampai CI/CD — kita berhasil bangun platform yang nggak cuma efisien, tapi juga resilient dan aman dari desainnya.

### Abis ini ngapain?
{{< typeit group=paragraph >}}
**Kayaknya makin banyak ngulik Cybersecurity dan AI . . .**
{{< /typeit >}}
{{< typeit group=paragraph >}}
**Dan semoga makin rajin nulis juga.**
{{< /typeit >}}