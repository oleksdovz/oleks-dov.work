# Kthena



Kthena (volcano-sh/kthena) — це Kubernetes-native платформа для model serving / LLM inference у продакшені, яка робить розгортання моделей і їхній lifecycle декларативними (через CRD) та інтегрується з типовими K8s-процесами.

![Diagram](https://github.com/volcano-sh/kthena/raw/main/docs/proposal/images/kthena-arch.svg)

Вона додає інтелектуальний request routing і механізми масштабування для inference-навантажень у кластерах, орієнтуючись на сценарії з високими вимогами до латентності та пропускної здатності. Окремий фокус — Prefill/Decode disaggregation: розділення compute-інтенсивного prefill та memory-/latency-чутливого decode для кращої утилізації апаратних ресурсів і досягнення SLO.

  ￼
Для цього Kthena описує ролі/групи serving’у (наприклад, prefill і decode як окремі ролі в serving group) та дозволяє керувати їхнім розміщенням і взаємодією як частиною платформи.

У релізах також підкреслюється topology awareness і role-level gang scheduling (на базі Volcano), що важливо для PD-схем, де критична мережна відстань між prefill та decode інстансами.  ￼

Документація містить “get started” та гіди, включно з прикладами для PD-дизагрегації (зокрема під NPU/Ascend сценарії), що робить репо корисним як для Platform/DevOps, так і для MLOps/AI Infra команд, які будують cloud-native inference слой



Github: [Kthena](https://github.com/volcano-sh/kthena)
