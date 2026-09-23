@startuml

!include 

https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml

LAYOUT_LEFT_RIGHT()

Container_Boundary(ranking, "Review Ranking Service") {

Component(snapshot, "Snapshot Builder","Java / SQL","Формирует неизменяемый snapshot данных операции")

Component(features, "Feature Builder","Python / SQL","Формирует временные, поведенческие и контрагентские признаки")

Component(model, "Ranking Model","Gradient Boosting / Learning-to-Rank","Рассчитывает priority score")

Component(calibration, "Score Calibration","Python","Калибрует значение score на контрольной выборке")

Component(policy, "Threshold & Abstention Policy","Java / configuration","Определяет зоны автоматической рекомендации и abstention")

Component(reasons, "Reason Builder","Python","Формирует ограниченный набор факторов, повлиявших на score")

Component(contract, "Output Contract Validator","Java","Проверяет схему, диапазоны и обязательные поля")
}

Container_Boundary(guardrail, "Decision Guardrail Service") {
Component(rules, "Mandatory Rule Engine","Java","Исполняет обязательные AML-правила")
Component(freshness, "Freshness & Coverage Check","Java","Проверяет полноту и актуальность признаков")
Component(conflict, "Conflict Resolver","Java","Обрабатывает конфликт ML-рекомендации и обязательных правил")
Component(fallback, "Fallback Router","Java","Переводит операцию в детерминированный или ручной сценарий")
}

Container(queue, "Review Queue","PostgreSQL / Kafka","Очередь ручной проверки")

Container(ui, "AML Review UI","Web","Рабочее место сотрудника")

Container(label, "Label & Control Sample Service","Java / SQL","Сохраняет результаты проверки и контрольной выборки")

Rel(snapshot, features, "Формирует snapshot")
Rel(features, model, "Передаёт признаки")
Rel(model, calibration, "priority score")
Rel(calibration, policy, "calibrated score")
Rel(policy, reasons, "Score + status")
Rel(reasons, contract, "Score + причины")

Rel(contract, rules, "Результат модели")
Rel(rules, freshness, "Проверка")
Rel(freshness, conflict, "Результат проверок")
Rel(conflict, fallback, "Итоговый статус")

Rel(fallback, queue, "REVIEW / FALLBACK / RULE_ONLY")
Rel(queue, ui, "Очередь")
Rel(ui, label, "Результат проверки")

@enduml
