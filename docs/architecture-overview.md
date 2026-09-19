# مخطط واجهة المستخدم والبيانات

```mermaid
flowchart LR
    U[المستخدم] --> UI[واجهة التطبيق
    Kotlin + Jetpack Compose]

    UI --> Auth[تسجيل الدخول
    المصادقة]
    UI --> FileOps[إدارة الملفات
    إنشاء • نسخ • نقل • حذف]
    UI --> Doc[قراءة التقرير
    والنصوص والوثائق]
    UI --> Search[بحث متقدم
    فلترة • تصنيف • فهرسة]

    Auth --> VM[ViewModel
    إدارة الحالة]
    FileOps --> VM
    Doc --> VM
    Search --> VM

    VM --> Repo[Repository
    منطق البيانات]

    Repo --> Local[(Room Database
    بيانات محلية)]
    Repo --> Model[نماذج البيانات
    DriveFile • DriveSection • SearchFilters]
    Repo --> Seed[InitialSeedData
    بيانات أولية]

    Local --> Files[(ملفات المستخدم
    المجلدات • المستندات • السلة)]
    Files --> UI

    Repo -. مزامنة مستقبلية .-> Cloud[Google Drive API
    أو خدمة سحابية مشابهة]
    Cloud --> Files

    UI --> Theme[Theme + Material 3
    الألوان • الخطوط • الأيقونات]
    UI --> Nav[Navigation
    التنقل بين الشاشات]
```

## شرح سريع

- المستخدم يتفاعل مع واجهات Compose عبر شاشات التطبيق.
- كل شاشة ترسل الأحداث إلى `ViewModel`.
- `ViewModel` يطلب البيانات من `Repository`.
- `Repository` يقرأ أو يكتب إلى `Room Database` محليًا.
- البيانات تُعرض في الواجهة فورًا بفضل حالة التطبيق التفاعلية.
- يمكن لاحقًا ربط المستودع بخدمة سحابية مثل Google Drive.

## هيكل التطبيق

```text
app/src/main/java/com/example/nizam/
├── MainActivity.kt
├── data/
│   ├── local/
│   │   ├── AppDatabase.kt
│   │   ├── DriveFileDao.kt
│   │   ├── DriveFileEntity.kt
│   │   └── InitialSeedData.kt
│   ├── model/
│   │   ├── DriveFile.kt
│   │   ├── DriveSection.kt
│   │   ├── SearchFilters.kt
│   │   └── NizamConstitutionalReport.kt
│   └── repository/
│       └── DriveRepository.kt
├── ui/
│   ├── components/
│   ├── screens/
│   └── theme/
├── viewmodel/
│   ├── DriveViewModel.kt
│   └── DriveViewModelFactory.kt
└── ...
```

## النطاق العملي لهذا المخطط

هذا المخطط مخصص لـ:
- عرض البنية الداخلية للتطبيق
- توضيح فصل الواجهة عن البيانات
- شرح كيف تتدفق المعلومات داخل التطبيق
- تقديم فكرة واضحة للعروض بدون تعقيد السحابة
