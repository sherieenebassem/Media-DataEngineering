# Media Data Engineering Platform

> **Project Charter — v0.1**  
> **Date:** 2026-11-17  
> **Status:** Initial agreement / living document  
> **Language:** Arabic explanations; English technical identifiers and code

## 1. Project Vision | رؤية المشروع

بناء منصة هندسة بيانات مركزية لإدارة بيانات مكتبات الأفلام والمسلسلات الموجودة على عدة سيرفرات يديرها صاحب المشروع، بدلًا من الاتصال بكل سيرفر واستخراج البيانات منه يدويًا في كل مرة.

المنصة تجمع بيانات المصادر، وتحفظ نسخها الأصلية، وتوحد شكلها، وتكتشف الإضافات والتغييرات، وتثري البيانات الوصفية عند الحاجة، ثم تتيح التحليل والتصدير حسب السيرفر والمحتوى والتصنيف بصيغتي **JSON** و**M3U** على الأقل.

الهدف ليس إعادة إنتاج أداة ETL القديمة، بل بناء نظام جديد من الصفر بهندسة برمجية قابلة للاختبار والصيانة والتوسع.

## 2. Core Principles | مبادئ التنفيذ

1. **Simple, focused tools:** اختيار مكتبات Python بسيطة ومتخصصة، وإضافة كل اعتماد عند وجود احتياج حقيقي.
2. **Modular Monolith:** تطبيق واحد منظم إلى مكونات مستقلة منطقيًا؛ لا Microservices مبكرًا.
3. **One service at a time:** نفهم ونصمم وننفذ ونختبر ونوثق المكون الحالي قبل الانتقال لغيره.
4. **Source preservation:** الاحتفاظ ببيانات المصدر الأصلية وإمكانية تتبع كل سجل إلى السيرفر وعملية السحب.
5. **Idempotent, resumable processing:** تصميم المزامنة لإعادة المحاولة والاستكمال من دون تكرار غير مقصود.
6. **No premature complexity:** لا Dashboards أو أدوات Orchestration أو خدمات إضافية قبل الاحتياج إليها.
7. **Evidence before assumptions:** فحص كود الأداة القديمة وعينات الاستجابات قبل اعتماد تفاصيل استخراج البيانات.

## 3. Functional Scope | المتطلبات الوظيفية المتفق عليها

### 3.1 Source connectivity

- دعم عدد متغير من السيرفرات، مع هوية مستقرة `server_id` لكل سيرفر.
- بيانات اتصال ديناميكية: `host`, `port`, `username`, `password` وغيرها حسب نوع الاتصال.
- دعم مصادر Xtream باستخدام `player_api.php`، وكذلك قوائم M3U عبر `get.php`.
- بناء **Source Adapters** مستقلة خلف واجهة استخراج موحدة، مع إمكانية إضافة أنواع أخرى مستقبلًا.
- عدم افتراض تطابق قدرات جميع السيرفرات أو بنية الاستجابات.

### 3.2 Extraction and raw storage

- استخراج تصنيفات الأفلام والمسلسلات والأعمال والمواسم والحلقات وروابط التشغيل والحقول المتاحة فعليًا.
- اكتشاف حقول إضافية مفيدة أثناء مراجعة الأداة القديمة وعينات السيرفرات.
- تنظيم البيانات الخام حسب السيرفر ونوع المحتوى وعملية/تاريخ السحب، مع عدم فقدان التصنيفات الأصلية.
- حفظ استجابات المصدر الأصلية وإتاحة إعادة المعالجة من دون طلبها مجددًا كل مرة.
- فصل **catalog metadata** عن نقل ملفات الفيديو؛ نقل ملفات الفيديو نفسها **قرار مفتوح** ولم يُعتمد ضمن المرحلة الأولى.

### 3.3 Batch and incremental synchronization

- تنفيذ سحب على دفعات مع تسجيل حالة كل عملية وإمكانية إعادة المحاولة.
- متابعة الجديد والمتغير في الأفلام والمسلسلات والمواسم والحلقات.
- دعم سيناريو مسلسل غير مكتمل تضاف له حلقات لاحقًا.
- اكتشاف تغير الروابط أو البيانات حتى لو لم يتغير عدد الحلقات.
- عدم اعتبار غياب سجل في استجابة واحدة دليلًا نهائيًا على الحذف.
- التحقق من حدود Pagination/Batching الفعلية لكل مصدر بدل افتراض أن API تدعم Pagination.

### 3.4 Standardization, catalog and enrichment

- توحيد الأسماء والحقول والأنواع مع الاحتفاظ بقيم المصدر الأصلية.
- التمييز بين **هوية العمل الفني** و**نسخة/إدراج العمل على سيرفر معين** و**ملف الوسائط أو رابط التشغيل**.
- اكتشاف التكرارات وربط العمل الواحد عبر السيرفرات من دون فقدان معلومات كل مصدر.
- إثراء البيانات الوصفية من مصادر مثل TMDB أو السينما.كوم **إذا سمحت الواجهات والشروط والتراخيص**.
- تسجيل حالات المطابقة غير المؤكدة للمراجعة بدل الدمج العشوائي.

### 3.5 Analytics, serving and exports

- عرض تقدم السحب والمزامنة، والأخطاء، وحالة المكتبات، واكتمال البيانات عبر Dashboards مستقبلية.
- تصدير البيانات النظيفة بحسب السيرفر أو نوع المحتوى أو التصنيف أو معايير اختيار أخرى.
- دعم **JSON** و**M3U** كمخرجات أساسية؛ ويمكن توليدهما من نفس الكتالوج.
- الحفاظ على الحقول الخاصة بالمصدر المطلوبة لإنتاج ملفات تصدير صحيحة.
- استخدام البيانات المركزية لتجهيز مكتبات جديدة أو إضافات لسيرفرات مستهدفة بعد التحقق من المتطلبات والصلاحيات.

## 4. Initial Technical Decisions | القرارات التقنية المبدئية

| Area | Choice | Status / Notes |
|---|---|---|
| Language | Python 3.12 | Adopted for current project environment |
| Environment & packaging | `uv`, `src/` package layout, Git | Initialized and import tested |
| Architecture | Modular Monolith | Adopted |
| Legacy ETL | Reference only, rebuild from scratch | Adopted |
| Source types | Xtream `player_api.php`, M3U `get.php` | Required |
| Operational state | SQLite | Preferred initial choice; revisit if concurrency demands |
| Object storage | MinIO | Planned central object storage |
| Raw formats | Source JSON / M3U | Preserve original payloads |
| Analytical formats | Parquet | Proposed for standardized/curated datasets |
| Analytical engine | DuckDB | Proposed |
| Orchestration | Prefect | Candidate for later stage, not first milestone |
| Transformations | Python; dbt if warranted | Not yet finalized |
| Management UI | Panel | Later, possibly multiple dashboards |
| Service API | FastAPI | Optional, introduce when required |
| Exports | JSON and M3U | Required |

**Architectural distinction:** MinIO is object storage; Parquet is a file format; DuckDB is an analytical engine; SQLite stores mutable operational state. These roles must not be conflated. Parquet is **not** the required delivery format.

## 5. Security and Operational Boundaries | الأمان

- عدم تخزين كلمات مرور السيرفرات أو الروابط الحاملة لبيانات دخول في Git أو ملفات توثيق عامة.
- عدم إظهار Credentials في Logs أو Dashboards أو صادرات تحليلية.
- تحديد طريقة إدارة الأسرار لاحقًا وفق بيئة التشغيل.
- مراعاة الصلاحيات وشروط استخدام مصادر الـ Metadata والمحتوى وروابط التشغيل.
- بيانات الاتصال المعروضة سابقًا كانت بيانات اختبار مؤقتة، ولا تُضمَّن في التوثيق.

## 6. First Workstation | المحطة الأولى

**Title:** Legacy ETL Review & Source Extraction Design

**Goal:** تصميم وتنفيذ أول Source Adapter على سيرفر واحد بعد فهم الكود القديم والبيانات الحقيقية.

**Sequence:**

1. استلام كود ETL القديم وفهم طريقة اتصاله بـ `player_api.php` ومخرجاته.
2. توثيق عيوبه والأسباب العملية لإعادة البناء.
3. حصر البيانات الحالية والبيانات الإضافية المرغوبة والمتاحة.
4. دراسة اختلافات `player_api.php` و`get.php`، وعينات الاستجابات والحالات الناقصة.
5. تعريف Data Contracts وحدود أول Adapter.
6. تصميم وتنفيذ واختبار أول Adapter فقط.
7. توثيق النتيجة والانتقال للمكون التالي بعد الاعتماد.

**Out of scope for the first workstation:** بناء Panel dashboards، أو نظام Matching كامل، أو خدمة تصدير شاملة، أو تشغيل جميع السيرفرات، أو نقل الفيديوهات، أو إدخال كل أدوات الـ Stack دفعة واحدة.

## 7. Current Local Project Setup | الوضع الحالي

```text
F:\me_projects\Media_DataEngineering\
├── docs\
├── legacy\
├── src\
│   └── media_data_engineering\
├── .venv\
├── .git\
├── pyproject.toml
├── .python-version
├── .gitignore
├── uv.lock
└── README.md
```

- Python: **3.12.6**.
- `uv sync`: succeeded.
- `uv run python -c "import media_data_engineering; print('Package OK')"`: succeeded.
- Git root: `F:/me_projects/Media_DataEngineering`.
- Project root is the **source of truth** for version-controlled documents and code.
- This file belongs at `docs/PROJECT_CHARTER.md`; a copy may be uploaded to the ChatGPT Project for discussion continuity.

## 8. Open Questions | قرارات مؤجلة

1. ما جميع الـ Endpoints والحقول التي توفرها السيرفرات الفعلية؟
2. ما التفاوت بين السيرفرات التي تدعم `player_api.php` والتي تتيح M3U فقط؟
3. ما قواعد الهوية والمطابقة الآمنة للأعمال والمواسم والحلقات؟
4. ما سياسة تكرار الفحص والتحديث، والتعامل مع الحذف والتغييرات؟
5. هل سيُخزَّن الفيديو نفسه أم الـ Metadata والروابط فقط؟
6. ما شكل JSON المطلوب للتصدير: نموذج موحد أم توافق مع سيرفر مستهدف؟
7. ما أفضل آلية لتخزين الأسرار وتشغيل Workers حسب بيئة النشر؟
8. ما مصادر الإثراء المسموح بها وشروط استخدامها؟
9. متى نحتاج Prefect أو dbt أو FastAPI أو Panel بالفعل؟

## 9. Documentation and Change Control | إدارة التوثيق

- **This charter:** الرؤية والنطاق والمبادئ والقرارات المبدئية.
- **Future requirements document:** المتطلبات التفصيلية بعد مراجعة الكود.
- **Future architecture decisions:** القرارات الهندسية وأسبابها والبدائل المرفوضة.
- **Future legacy review:** ملاحظات تحليل الكود القديم وعينات المخرجات.
- لا نعتبر الاقتراحات التقنية حقائق نهائية إلا بعد مناقشتها واعتمادها.
- كل تغيير جوهري يُسجل بإصدار وتاريخ وسبب واضح.

## 10. Immediate Next Action | الخطوة التالية

1. حفظ هذه الوثيقة في `docs/PROJECT_CHARTER.md` محليًا.
2. رفع نسخة إلى ملفات مشروع ChatGPT **Media-DataEngineering** عند الرغبة.
3. فتح محادثة بعنوان **المحطة 01 — Legacy ETL Review & Source Extraction Design**.
4. مشاركة الكود القديم، ثم شرح ما لا يعجب صاحب المشروع فيه، وما ينقصه، وما البيانات الإضافية المطلوبة.

---

*Version 0.1 is a project charter, not a finalized requirements specification or architecture blueprint.*
