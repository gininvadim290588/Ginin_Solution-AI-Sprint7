# Задание 3. Проектирование AI-сервиса для флагманской инициативы

## 1. Флагманская инициатива

**Инициатива №1 — автоматическое выявление подозрительных операций.**

ML не принимает юридическое решение о признании операции подозрительной и не выполняет автоматическую блокировку. Целевой сценарий: ML ранжирует операции по приоритету ручной проверки и формирует объяснение результата.

Обязательные AML-правила, внешние сигналы и ответственность сотрудников банка сохраняются.

## 2. C4 Container
Диаграмма представлена в папке Diagrams

## 3. Основной поток
```
Система операций
       |
       v
Operation Ingestion
       |
       v
Feature Service <---- DWH / внешние сигналы
       |
       v
Review Ranking Service
       |
       v
Decision Guardrail Service
       |
       v
Review Queue
       |
       v
AML Review UI
       |
       v
AML-аналитик
       |
       +----> Label & Control Sample Service
```

Обязательные AML-правила проходят через deterministic-контур независимо от ML.

## 4. Component Diagram для Review Ranking Service
Диаграмма представлена в папке Diagrams

## 5. Decision Guardrail

Состав:
- **Mandatory Rule Engine** — обязательные AML-правила;
- **Freshness & Coverage Check** — свежесть и полнота;
- **Conflict Resolver** — разрешение конфликтов;
- **Fallback Router** — rule-only / ручная проверка / резервный сервис.

При конфликте приоритет имеют обязательные deterministic-правила.

## 6. Output Contract

```json
{
  "operation_id": "string",
  "decision_time": "ISO-8601",
  "priority_score": 0.0,
  "rank": 123,
  "reasons": [
    {
      "code": "string",
      "value": "string",
      "source": "feature_id"
    }
  ],
  "model": {
    "name": "string",
    "version": "string"
  },
  "feature_snapshot_id": "string",
  "source_refs": [
    {
      "source_id": "string",
      "type": "transaction|signal|aggregate"
    }
  ],
  "mandatory_rule_flags": [],
  "status": "REVIEW|FALLBACK|RULE_ONLY",
  "abstained": false
}
```

При отсутствии критических данных ML не генерирует фиктивное значение: операция переводится в fallback / rule-only / ручную проверку.

## 7. Режим обработки

Основной режим — **asynchronous**. ML не должен блокировать проведение платежа.

**Batch** используется для исторического пересчёта, обучения, backtesting, regression testing и drift analysis.

Идемпотентность:

```text
(operation_id, feature_snapshot_id, model_version)
```

## 8. Уровень автоматизации

**Recommendation / human-in-the-loop.**

ML рекомендует приоритет и формирует объяснение. AML-аналитик принимает окончательное решение.

## 9. Fallback

Срабатывает при недоступности ML, timeout, недостатке/устаревании данных, OOD, abstention, ошибке схемы, отсутствии source reference, конфликте источников.

```text
ML недоступен
     |
     +--> Rule-only
     |
     +--> Ручная проверка
     |
     +--> Резервный сервис
```

## 10. Мониторинг

### Data Quality
- completeness;
- freshness;
- schema violations;
- missing values.

### Model Quality
- Recall@K;
- Precision@K;
- Lift;
- scenario recall;
- calibration;
- drift.

### Operational
- latency;
- error rate;
- throughput;
- queue size;
- fallback rate.

### Human
- время проверки;
- доля принятых/отклонённых рекомендаций;
- нагрузка аналитиков.

### Business
- количество проверок;
- углублённые расследования;
- подтверждённые случаи;
- приостановки;
- снятые приостановки;
- жалобы.

## 11. Безопасность

Trust boundaries:
- T1 — системы операций → AI-контур;
- T2 — внешние AML/KYC-сигналы → AI-контур;
- T3 — AI-контур → AML UI;
- T4 — AML UI → пользователь;
- T5 — AI-контур → DWH/Feature Store;
- T6 — контур разметки/обучения.

Требования:
- минимально необходимые права;
- аудит доступа;
- шифрование;
- журналирование;
- versioning моделей;
- контроль изменений;
- разделение production и training data.

Клиентские документы и текстовые поля рассматриваются как **untrusted content**.

## 12. Инициатива №2 — OCR документов

```text
Document Intake
       |
       v
OCR & Document Classification
       |
       v
Field Extraction
       |
       v
Extraction Validator
       |
       +----> Human Review UI
       |
       v
Application Data
```

Низкая уверенность и критические ошибки передаются человеку.

## 13. Инициатива №18 — финансовые показатели

```text
Document Intake
       |
       v
Layout & Table Parsing
       |
       v
Financial Extraction
       |
       v
Normalization & Validation
       |
       +----> Analyst Review
       |
       v
Financial Profile
```

Пример результата:

```json
{
  "metric_name": "revenue",
  "value": 1234567,
  "period": "2025",
  "unit": "RUB",
  "source": "document/page/table",
  "confidence": 0.96
}
```

Детерминированная часть выполняет нормализацию единиц и дат, обработку знаков, арифметические проверки и контроль согласованности. Низкая уверенность или критический конфликт → ручная проверка.

## 14. Итоговая архитектурная позиция

Для флагманской инициативы предлагается **AI ranking service с human-in-the-loop**, а не автономный классификатор.

Ключевые принципы:
1. ML не заменяет обязательный AML-контроль.
2. ML не принимает юридическое решение.
3. Исторические расследования — слабая разметка, а не абсолютная истина.
4. Нужна независимая вероятностная контрольная выборка.
5. Каждый ML-результат должен быть воспроизводим.
6. Обязательны versioning модели и feature snapshot.
7. При критической ошибке используется deterministic fallback.
8. Клиентские документы и текстовые поля считаются недоверенным входом.
9. Критические решения остаются под контролем сотрудника.
10. Production допускается только после успешного пилота.
11. 
