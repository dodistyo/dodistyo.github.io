---
weight: 1000
title: "Kubernetes Gateway API"
date: 2025-07-05T00:00:00+07:00
lastmod: 2025-07-05T00:00:00+07:00
draft: false
author: "Dodi"
description: "Panduan santai buat ngeakses layanan Kubernetes pakai NodePort, LoadBalancer, Ingress, dan Gateway API versi terbaru."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"
tags: ["Platform", "Kubernetes", "Gateway API", "Menengah", "north/south traffic"]
categories: ["Kubernetes", "Networking", "Gateway API"]
lightgallery: false
---

Masih inget nggak masa-masa awal waktu lu bingung gimana caranya aplikasi lu bisa diakses dari luar cluster Kubernetes?

<!--more-->

Bisa berhasil bikin dan nyalain cluster Kubernetes, terus nge-deploy aplikasi yang udah di-containerize itu rasanya puas banget.

Semua jalan lancar, semua pod statusnya `running`. lu jadi semangat banget, tapi terus mikir: “Terus, sekarang gimana dong?”

Aplikasi lu udah jalan.

Tapi gimana cara akses service aplikasinya?

Akhirnya lu mulai selam ke dokumentasi Kubernetes, dan baca-baca soal service dan tipe-tipenya.

## Tipe-tipe Service diKubernetes

Secara default, Kubernetes service pake tipe `ClusterIP`. Jadi kalau lu nggak nulis `type` di manifest service-nya, Kubernetes bakal anggap itu sebagai `ClusterIP`.

Apa sih kerjaannya `ClusterIP`?

 - Bikin IP internal yang stabil buat servicenya.
 - Cuma bisa diakses dari dalam cluster (misalnya pod atau node lain), nggak bisa dari luar.

Lalu lu nemuin tipe service **NodePort**, salah satu cara buat nge-expose aplikasi biar bisa diakses dari luar cluster. Tapi waktu lu setup `NodePort`, ternyata dia buka port yang sama di semua node di cluster yang mana nggak terlalu oke. Apalagi lu sekarang harus mikirin load balancer per node, duh...

Lanjut, lu baca tentang tipe service **LoadBalancer** yang ternyata lebih ideal. Soalnya dia cuma buka satu endpoint aja dan nggak buka banyak port di tiap node.

Tapi fitur ini cuma otomatis jalan kalau lu pake layanan Kubernetes managed kayak GKE, EKS, AKS, dll. Dan tentu aja, ada biayanya.

Kalau lu pake Kubernetes yang di-setup sendiri (on-premise)? lu butuh tool tambahan kayak **MetalLB** buat ngatur load balancer di Layer 2.

## Kubernetes Ingress

Tipe service `LoadBalancer` udah bagus, tapi ternyata ada cara yang lebih canggih buat nge-expose service lu. pada dasarnya sama aja kaya lu pake `LoadBalancer` plus pakein reverse proxy juga di depan service-service lu.

Yeay! lu mulai kenal deh sama Kubernetes ingress controller, dan lu mulai coba deploy controller pertamalu.

Ingress controller yang paling umum dipake biasanya **ingress-nginx** (jangan sampai ketuker sama **nginx-ingress**, soalnya itu dua proyek yang berbeda). Ada juga controller lain kayak **emissary-ingress**, proyek yang udah graduate dari CNCF, yang pake envoy proxy sebagai dataplane dan punya fitur-fitur lebih advance yang layak lu pertimbangkan juga.

## Batasan Kubernetes Ingress

Awalnya semua lancar sampai lu ketemu sama keterbatasan ingress di Kubernetes.

Buat banyak workload, resource Ingress standar bareng ingress controller kayak ingress-nginx udah cukup banget.

Tapi makin lama kebutuhan lu makin kompleks, dan lu mulai ngadepin batasannya

**Routing L7 yang Terbatas**

Resource Ingress cuma support routing berdasarkan path dan host di HTTP.

Tapi kalau lu butuh routing berdasarkan header, cookie, atau aturan kompleks lainnya, biasanya perlu anotasi custom, CRD, atau bahkan controller yang beda.

**Gak Support TCP/UDP Secara Native**

Ingress standar cuma ngatur traffic HTTP dan HTTPS. Kalau lu perlu expose service TCP atau UDP, lu harus nambah konfigurasi khusus. Beberapa controller emang bisa handle dengan config map tambahan, tapi itu bukan bagian dari spesifikasi resmi Ingress.

**Anotasi dan Fitur Tergantung Controller**

Setiap ingress controller punya anotasi sendiri-sendiri. Jadi lu jadi tergantung sama controller itu. Kalau nanti lu mau ganti controller, manifest YAML lu kemungkinan harus disesuaikan.

**Observability dan Policy Control yang Terbatas**

Secara default, Ingress cuma ngasih metrik basic kayak jumlah request dan error. Tapi kalau lu pengen observability lebih dalam, rate limiting, autentikasi, dan kontrol policy, lu butuh tool tambahan atau versi enterprise.

**Gak Ada Standar Integrasi dengan Load Balancer**

Walaupun tipe service `LoadBalancer` biasanya otomatis bikin cloud load balancer, Ingress sendiri nggak punya standar untuk integrasi LB eksternal. Beberapa controller emang bantuin soal ini, tapi itu tergantung masing-masing controller.

Kalau lu udah mentok dengan ingress, tenang aja Kubernetes udah berkembang dan sekarang ada solusi barunya!

## Kenalan Dengan Kubernetes Gateway API

Sekarang kenalan yuk sama **Kubernetes Gateway API** (jangan salah paham sama API Gateway, penjelasannya bisa lu baca di [sini](https://gateway-api.sigs.k8s.io/#whats-the-difference-between-gateway-api-and-an-api-gateway)).

Gateway API ini adalah cara standar yang lebih modern buat nge-expose service di dalam cluster Kubernetes.

Gateway API adalah cara yang lebih bener, powerful, dan fleksibel buat ngatur routing network ke dalam cluster.

API ini dibuat buat nutupin kekurangan yang ada di Ingress:

- Routing canggih yang distandarisasi (termasuk header, query param, dll)
- Dukungan bawaan buat TCP, UDP, dan bahkan TLS termination all in one API
- Misahin infrastruktur (GatewayClass, Gateway) dari aturan routing (HTTPRoute, TCPRoute, dll)
- Spesifikasi netral jadi manifest lu lebih portable dan gak tergantung vendor

Banyak controller populer sekarang udah support Gateway API. lu bisa cek daftarnya di [sini](https://gateway-api.sigs.k8s.io/implementations/#gateway-controller-implementation-status).

## Cooking Time!

### Oprekan yang gue pake:

- Kubernetes distro: [KIND](https://kind.sigs.k8s.io/) cluster
- GitOps tools: [FluxCD](https://fluxcd.io/)
- LoadBalancer: [MetalLB](https://metallb.io/)
- Gateway Controller: [Envoy Gateway](https://gateway.envoyproxy.io/)
- Certificate Manager (for https): [Cert Manager](https://cert-manager.io/)
- Source Code Repository: [dodistyo/kivotos](https://github.com/dodistyo/kivotos.git)

### Pre-konfigurasi

Pastikan semua komponen platform udah dideploy dan dikonfigurasi (metallb, envoy gateway, dan cert-manager).

Gue juga udah setup hostname custom `app.me` biar langsung ngarah ke service Gateway API, jadi nanti kita bisa akses pake `app.me` buat keperluan testing.

### Deploymen Resource

Sekarang kita deploy [contoh service](https://github.com/dodistyo/kivotos/tree/main/base/application/demo/services) yang terdiri dari service TCP/UDP, HTTP, dan gRPC.

![Manifest](/demo-services.png)

Sekarang kita deploy [resources](https://github.com/dodistyo/kivotos/blob/main/base/platform/setup/gateway-api.yaml) Gateway API kita. Kita perlu setup `GatewayClass` dan `Gateway` sebagai konfigurasi platform Gateway API kita.

Kita apply ini di namespace `platform`.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy-gateway
  namespace: platform
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: main-gateway
  namespace: platform
spec:
  gatewayClassName: envoy-gateway
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
    - name: https
      protocol: HTTPS
      port: 443
      allowedRoutes:
        namespaces:
          from: All
      tls:
        mode: Terminate
        certificateRefs:
          - kind: Secret
            name: selfsigned-tls
            group: ""
    - name: tcp
      protocol: TCP
      port: 8080
      allowedRoutes:
        kinds:
        - kind: TCPRoute
        namespaces:
          from: All
    - name: udp
      protocol: UDP
      port: 8081
      allowedRoutes:
        kinds:
        - kind: UDPRoute
        namespaces:
          from: All
    - name: grpc
      protocol: HTTPS
      port: 50051
      allowedRoutes:
        namespaces:
          from: All
      tls:
        mode: Terminate
        certificateRefs:
          - kind: Secret
            name: selfsigned-tls
            group: ""
```

Sekarang kita apply resource buat mapping dan [routing](https://github.com/dodistyo/kivotos/blob/main/base/application/demo/routes.yaml) service demo kita.

- **ReferenceGrant**: Ini kita butuhin buat ngasih akses ke Gateway API kita, soalnya resource platform Gateway API-nya ada di namespace yang beda sama resource buat mapping dan routing service-nya.
- **TCPRoute & UDPRoute**: Dipake buat routing ke server TCP/UDP kita.
- **Dua HTTPRoute**: Satu buat routing ke HTTP server, satunya lagi cuma buat redirect ke HTTPS.
- **GRPCRoute**: Buat routing ke server gRPC kita.


```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: ReferenceGrant
metadata:
  name: allow-demo-to-platform
  namespace: platform
spec:
  from:
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
      namespace: demo
  to:
    - group: gateway.networking.k8s.io
      kind: Gateway
      name: main-gateway
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-to-https-redirect
  namespace: demo
spec:
  parentRefs:
  - name: main-gateway
    sectionName: http
    namespace: platform
  hostnames:
  - "app.me"
  rules:
  - filters:
    - type: RequestRedirect
      requestRedirect:
        scheme: https
        statusCode: 301
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: https-server-route
  namespace: demo
spec:
  parentRefs:
  - name: main-gateway
    namespace: platform
    sectionName: https
  hostnames:
  - "app.me"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /http-echo
    backendRefs:
    - name: http-echo
      namespace: demo
      port: 80
---
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: TCPRoute
metadata:
  name: tcp-app-route
  namespace: demo
spec:
  parentRefs:
  - name: main-gateway
    namespace: platform
    sectionName: tcp
  rules:
  - backendRefs:
    - name: tcp-udp-service
      namespace: demo
      port: 8080
---
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: UDPRoute
metadata:
  name: udp-app-route
  namespace: demo
spec:
  parentRefs:
  - name: main-gateway
    namespace: platform
    sectionName: udp
  rules:
  - backendRefs:
    - name: tcp-udp-service
      namespace: demo
      port: 8081
---
apiVersion: gateway.networking.k8s.io/v1
kind: GRPCRoute
metadata:
  name: grpc-route
  namespace: demo
spec:
  parentRefs:
  - name: main-gateway
    namespace: platform
    sectionName: grpc
  hostnames:
  - "app.me"
  rules:
  - backendRefs:
    - name: grpc-hello-service
      namespace: demo
      port: 50051
```

Yuk jajalin hasilnya!

- TCP/UDP server tes make netcat

  ![Shell](/demo-tcp-udp-test.png)

- HTTP server tes.

  Liat cuy...

  Langsung ke redirect ke https kan? yaa walaupun sertifikatnya self-signed

  Tapi santai aja, cuma buat tes kok.

  ![Shell](/demo-http-test.png)

- gRPC server tes make grpcurl

  ![Shell](/demo-grpc-test.png)

## Bacaan Lebih Lanjut

- [https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)
- [https://gateway-api.sigs.k8s.io/](https://gateway-api.sigs.k8s.io/)
- [https://gateway-api.sigs.k8s.io/implementations/#service-mesh-implementation-status](https://gateway-api.sigs.k8s.io/implementations/#service-mesh-implementation-status)
