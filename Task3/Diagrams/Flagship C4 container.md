@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_LEFT_RIGHT()

Person(analyst, "Сотрудник финансового мониторинга","Проводит проверку операции и принимает итоговое решение")

System_Ext(tx, "Транзакционные системы","Платежи, счета, операции клиентов")

System_Ext(ext, "Внешние источники сигналов","Внешние AML/KYC-сигналы и справочная информация")

System_Ext(dwh, "DWH / Feature Store","Исторические операции и агрегированные признаки")

System_Boundary(ai, "AI / AML-контур") {

Container(ingest, "Operation Ingestion","Java / SQL","Получение операции и формирование snapshot доступных данных")

Container(feature, "Feature Service","Python / SQL","Формирование признаков на момент принятия решения")

Container(rank, "Review Ranking Service","Python / ML","Расчёт priority score и ранжирование операций")

Container(guard, "Decision Guardrail Service","Java","Калибровка, пороги, обязательные правила, проверки качества и fallback")

Container(queue, "Review Queue","PostgreSQL / Kafka","Очередь операций в порядке приоритета проверки")

Container(ui, "AML Review UI","Web","Рабочее место сотрудника финансового мониторинга")

Container(label, "Label & Control Sample Service","Java / SQL","Сбор результатов ручной проверки и вероятностной контрольной выборки")

Container(monitor, "Monitoring","Prometheus / Grafana","Мониторинг данных, модели, очереди и бизнес-метрик")
}

Rel(tx, ingest, "Передаёт операции")
Rel(ext, ingest, "Передаёт внешние сигналы")
Rel(dwh, feature, "Исторические данные и агрегаты")
Rel(ingest, feature, "Передаёт snapshot")
Rel(feature, rank, "Передаёт признаки")
Rel(rank, guard, "Передаёт score")
Rel(guard, queue, "Передаёт проверенные результаты")
Rel(queue, ui, "Показывает очередь")
Rel(ui, analyst, "Рабочее место")
Rel(analyst, label, "Результат проверки")
Rel(label, monitor, "Метрики и labels")
Rel(rank, monitor, "Метрики модели")
Rel(feature, monitor, "Метрики качества данных")
Rel(queue, monitor, "Метрики очереди")

@enduml
