# AsciiArtify — Concept: локальний Kubernetes для PoC

## 1. Вступ

| Інструмент | Призначення |
|---|---|
| **minikube** | Офіційний проєкт Kubernetes SIG. Локальний кластер (зазвичай один вузол) у VM або контейнері; багато драйверів (docker, podman, kvm2, virtualbox, hyperkit). |
| **kind** | Kubernetes IN Docker. Вузли кластера — контейнери. Створений для тестування самого Kubernetes та CI. |
| **k3d** | Обгортка над **k3s** (легкий дистрибутив Rancher/SUSE, не RKE) у Docker-контейнерах. Швидкий старт, мультивузлові кластери. |

**Ризик ліцензування Docker:** Docker Desktop платний для компаній >250 працівників або з доходом >$10 млн. Для малого стартапу безкоштовно, але при зростанні це ризик. Docker Engine на Linux (Apache 2.0) безкоштовний. Альтернатива — **Podman** (daemonless, rootless, Apache 2.0): підтримується minikube (`--driver=podman`), kind (`KIND_EXPERIMENTAL_PROVIDER=podman`), k3d має лише часткову/експериментальну підтримку через podman socket.

## 2. Характеристики

| Характеристика | minikube | kind | k3d |
|---|---|---|---|
| ОС | Linux, macOS, Windows | Linux, macOS, Windows | Linux, macOS, Windows |
| Архітектури | amd64, arm64, ppc64le, s390x, riscv64 | amd64, arm64 | amd64, arm64 |
| Середовище | VM / контейнер / bare metal | Docker/Podman/nerdctl контейнери | Docker контейнери (k3s) |
| Дистрибутив K8s | upstream kubeadm | upstream kubeadm | k3s (полегшений) |
| Мультивузловість | є (`--nodes`), повільніше | є (конфіг YAML) | є (`--servers/--agents`), дуже швидко |
| Автоматизація | CLI, профілі | CLI + YAML-конфіг, добре для CI | CLI + YAML-конфіг, добре для CI |
| LoadBalancer | `minikube tunnel` | MetalLB вручну | вбудований (klipper + traefik, `-p` port mapping) |
| Аддони/моніторинг | `minikube addons` (dashboard, metrics-server, ingress, registry…) | мінімум, усе вручну | traefik, metrics-server з k3s; решта вручну |
| Локальний registry | addon | вручну | `--registry-create` |
| Dashboard | `minikube dashboard` | ні | ні (Lens/k9s) |

## 3. Переваги та недоліки

| | Переваги | Недоліки |
|---|---|---|
| **minikube** | Найбагатша функціональність (addons, dashboard); найкраща документація; багато драйверів, у т.ч. Podman; максимально близький до upstream | Найважчий і найповільніший старт; більше ресурсів; масштабування мультивузлових кластерів громіздке |
| **kind** | Чистий upstream K8s; стабільний; чудовий для CI; підтримує HA-конфіги; Podman підтримується | Немає addons/dashboard; ingress/LB налаштовувати вручну; образи потрібно завантажувати `kind load` |
| **k3d** | Дуже швидкий старт (секунди); легкий; вбудовані LB, ingress, registry; прості мультивузлові кластери | k3s ≠ повністю upstream (деякі компоненти вирізані/замінені); підтримка Podman обмежена; менша спільнота за minikube |

## 4. Демонстрація (рекомендований інструмент: k3d)

> Нижче наведено **очікуваний вивід** команд (ілюстративний, зібраний за документацією k3d; не запис реального запуску).

```console
$ curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
$ k3d cluster create asciiartify --servers 1 --agents 2 -p "8080:80@loadbalancer"
INFO[0000] Created cluster 'asciiartify'

$ kubectl get nodes
NAME                       STATUS   ROLES                  AGE   VERSION
k3d-asciiartify-server-0   Ready    control-plane,master   30s   v1.x.x+k3s1
k3d-asciiartify-agent-0    Ready    <none>                 25s   v1.x.x+k3s1
k3d-asciiartify-agent-1    Ready    <none>                 25s   v1.x.x+k3s1

$ kubectl create deployment hello --image=nginxdemos/hello --replicas=2
$ kubectl expose deployment hello --port=80
$ kubectl create ingress hello --rule="/=hello:80"

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
hello-xxxxxxxxxx-aaaaa   1/1     Running   0          15s
hello-xxxxxxxxxx-bbbbb   1/1     Running   0          15s

$ curl -s http://localhost:8080 | grep -i "<title>"
<title>Hello World</title>

$ kubectl scale deployment hello --replicas=4
$ kubectl get pods --no-headers | wc -l
4

$ k3d cluster delete asciiartify
```

## 5. Висновки та рекомендації

| Критерій | minikube | kind | k3d |
|---|---|---|---|
| Швидкість старту | ★★ | ★★★ | ★★★★★ |
| Ресурси | ★★ | ★★★ | ★★★★ |
| Простота | ★★★★ | ★★★ | ★★★★★ |
| Функціональність «з коробки» | ★★★★★ | ★★ | ★★★★ |
| Близькість до upstream | ★★★★★ | ★★★★★ | ★★★ |
| Підтримка Podman | ★★★★★ | ★★★★ | ★★ |

- **k3d — рекомендація для PoC**: швидкий, легкий, готові LB/ingress/registry, зручно показувати масштабування кількох вузлів.
- **minikube** — якщо потрібні dashboard/addons, VM-ізоляція або Podman як основний рантайм (відмова від Docker Desktop).
- **kind** — для CI-пайплайнів GitHub Actions і перевірки сумісності з чистим upstream.
- **Docker/ліцензії:** на Linux використовувати Docker Engine, на macOS/Windows при зростанні команди — перейти на Podman + minikube/kind.
