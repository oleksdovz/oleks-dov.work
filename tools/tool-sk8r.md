# sk8r

[sk8r](https://github.com/mvklingeren/sk8r) — це open-source “клон” Kubernetes Dashboard з сучасним UI, який дає швидкий огляд і керування ресурсами кластера:
- перегляд/керування Pods, Deployments, Services,
- стрімінг логів у реальному часі,
- інтерактивний shell у Pod та (опційно) метрики через Prometheus.

---
Розгортання максимально прямолінійне: автор пропонує kubectl apply маніфестів (RBAC + Deployment + Service), а для метрик очікується доступний Prometheus (наприклад з kube-prometheus-stack).

---
Проєкт добре підходить як легкий “операторський” дашборд для dev/stage кластерів і як база для кастомних внутрішніх панелей.
