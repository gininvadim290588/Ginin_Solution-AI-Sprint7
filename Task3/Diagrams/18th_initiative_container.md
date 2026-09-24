@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_LEFT_RIGHT()

Person(underwriter, "Андеррайтер","Проверяет финансовые показатели")

System_Ext(app, "Credit Application","Кредитная заявка")

System_Ext(docs, "Бухгалтерские документы","Отчётность, банковские выписки и другие документы")

System_Boundary(ai, "Financial Extraction AI") {

    Container(intake, "Document Intake","Java","Принимает документы и сохраняет provenance")

    Container(parser, "Layout & Table Parser","Python / OCR","Определяет структуру документа и таблиц")

    Container(extraction, "Financial Extraction","ML","Извлекает финансовые показатели")

    Container(normalization, "Normalization & Validation","Java / Rules","Нормализует единицы, периоды, знаки и выполняет проверки")

    Container(review, "Analyst Review","Web","Проверка низкоуверенных и противоречивых значений")

    Container(profile, "Financial Profile","PostgreSQL","Хранит нормализованный финансовый профиль клиента")
}

Rel(app, intake, "Передаёт документы")
Rel(docs, intake, "Передаёт документы")
Rel(intake, parser, "Документ")
Rel(parser, extraction, "Структура и таблицы")
Rel(extraction, normalization, "Извлечённые показатели")
Rel(normalization, profile, "Проверенные показатели")
Rel(normalization, review, "Исключения")
Rel(review, profile, "Подтверждённые показатели")
Rel(profile, underwriter, "Финансовый профиль")

@enduml
