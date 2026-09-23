@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_LEFT_RIGHT()

Person(underwriter, "Андеррайтер","Проверяет кредитную заявку и документы")

System_Ext(app, "Credit Application","Кредитная заявка и прикреплённые документы")

System_Boundary(ai, "Document AI") {

Container(intake, "Document Intake","Java","Принимает документ и проверяет его технические характеристики")
Container(ocr, "OCR & Document Classification","ML / OCR","Распознаёт текст и определяет тип документа")
Container(extraction, "Field Extraction","ML","Извлекает поля документа")
Container(validation, "Extraction Validator","Java / Rules","Проверяет формат, обязательность и согласованность полей")
Container(review, "Human Review UI","Web","Передаёт низкоуверенные или критические поля человеку")
Container(data, "Application Data","PostgreSQL","Хранит нормализованные данные заявки")
}

Rel(app, intake, "Передаёт документы")
Rel(intake, ocr, "Документ")
Rel(ocr, extraction, "OCR-текст + тип документа")
Rel(extraction, validation, "Извлечённые поля")
Rel(validation, data, "Валидированные поля")
Rel(validation, review, "Низкая уверенность / ошибка")
Rel(review, data, "Подтверждённые значения")
Rel(data, underwriter, "Данные заявки")

@enduml
