# kubeplugin

Плагін kubectl: статистика CPU/Memory

## Встановлення
    sudo cp kubeplugin /usr/local/bin/kubectl-kubeplugin
    kubectl plugin list

## Використання
    kubectl kubeplugin [namespace] [pods|nodes]

За замовчуванням: `kube-system pods`.

    kubectl kubeplugin kube-system pods

Вивід: `Resource, Namespace, Name, CPU, Memory`
