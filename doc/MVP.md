# MVP: AsciiArtify + ArgoCD

Репозиторій застосунку: https://github.com/zero23-eu/go-demo-app (fork від https://github.com/den-vasyliev/go-demo-app)

## Крок 1. Створити кластер

```
k3d cluster create argo
kubectl get nodes
```

## Крок 2. Встановити ArgoCD

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd
```

Чекаємо, поки всі поди будуть Running.

## Крок 3. Відкрити UI

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Відкрити https://localhost:8080 і прийняти самопідписаний сертифікат.

## Крок 4. Отримати пароль адміна

Встановити ArgoCD CLI:

```
brew install argocd
```

Отримати пароль:

```
argocd admin initial-password -n argocd
```

Увійти: логін admin, пароль з команди вище.

## Крок 5. Створити застосунок

В UI натиснути New App і заповнити:

- Application name: demo
- Project: default
- Sync policy: Manual
- Auto-create namespace: увімкнути
- Repository URL: https://github.com/zero23-eu/go-demo-app
- Revision: main
- Path: helm
- Cluster: https://kubernetes.default.svc
- Namespace: demo

Натиснути Create.

## Крок 6. Синхронізувати

Застосунок спочатку OutOfSync. Натиснути Sync, обрати ресурси і Synchronize. Дочекатися статусу Synced / Healthy.

[ArgoCD-fork-demo-local.mp4](ArgoCD-fork-demo-local.mp4)
[app-demo.mp4](app-demo.mp4)
