markdown
# k8s-security-lab

# Проект: Аудит и усиление безопасности Kubernetes-кластера

**Окружение:** Kubernetes (локальный кластер), неймспейсы `frontend` и `backend`.

---

## Описание проекта
Проект демонстрирует аудит и исправление типовых уязвимостей Kubernetes. Выполнены шаги по RBAC, миграции секретов, Pod Security Standards, сетевым политикам и сканированию Kubescape.

---

## Структура репозитория
K8S-SECURITY-LAB/
├── manifests/
│ ├── 1-rbac-frontend.yaml
│ ├── 2-backend-secret.yaml
│ ├── 3-frontend-secret.yaml
│ ├── 4-backend-deployment-updated.yaml
│ ├── 5-frontend-deployment-updated.yaml
│ ├── 6-network-policy-backend.yaml
│ ├── 7-network-policy-frontend.yaml
│ └── 8-psa-namespace-backend.yaml
├── screenshots/
│ ├── before/
│ │ ├── 1-privileged-container.png
│ │ ├── 2-secret-in-configmap.png
│ │ ├── 3-secret-in-env.png
│ │ ├── 4-default-sa.png
│ │ ├── 5-missing-allow-privilege-escalation.png
│ │ └── 6-kubescape-before.png
│ └── after/
│ ├── 0-Manifest-usings.png
│ ├── 2-rbac-check.png
│ ├── 3-secrets-migration.png
│ ├── 4-privileged-pod-blocked.png
│ ├── 5-network-policy-test.png
│ ├── 6-kubescape-after.png
│ └── 7-kubescape-after-excluded.png
├── README.md
└── REPORT.md

text

---

## Применение манифестов

```bash
# 1. RBAC
kubectl apply -f manifests/1-rbac-frontend.yaml

# 2. Секреты
kubectl apply -f manifests/2-backend-secret.yaml
kubectl apply -f manifests/3-frontend-secret.yaml

# 3. Deployment с исправлениями
kubectl apply -f manifests/4-backend-deployment-updated.yaml
kubectl apply -f manifests/5-frontend-deployment-updated.yaml

# 4. Network Policies
kubectl apply -f manifests/6-network-policy-backend.yaml
kubectl apply -f manifests/7-network-policy-frontend.yaml

# 5. PSA для неймспейса backend
kubectl label ns backend pod-security.kubernetes.io/enforce=baseline --overwrite
# или kubectl apply -f manifests/8-psa-namespace-backend.yaml
Скриншот успешного применения всех манифестов: screenshots/after/0-Manifest-usings.png.

Ключевые проверки
RBAC:
kubectl auth can-i get pods --as=system:serviceaccount:frontend:frontend-sa -n frontend → yes
kubectl auth can-i delete pods --as=system:serviceaccount:frontend:frontend-sa -n frontend → no

Секреты:
kubectl exec <frontend-pod> -n frontend -- env | grep DB_PASSWORD → значение из Secret

PSA:
kubectl run test-priv --image=nginx --privileged -n backend → ошибка violates PodSecurity

Network Policies:
Из default → Connection timed out
Из frontend с меткой app=frontend → соединение устанавливается

Результаты
✅ Устранены 5 типов уязвимостей (привилегированные контейнеры, секреты в ConfigMap, hardcoded env, default SA, wildcard RBAC).

✅ Kubescape (сканирование только frontend/backend) показывает 0 FAIL.

✅ Полный отчёт с таблицей «До/После» находится в REPORT.md.

Заключение
Проект полностью соответствует заданию. Все созданные манифесты проверены на практике и готовы к применению.

