# المخطط المعماري التنفيذي لتطبيق النظام

> مخطط مناسب للعروض التقديمية يوضح تجربة المستخدم، طبقات التطبيق، وخيارات الاستضافة السحابية. أسماء الخدمات السحابية أدناه **تصميم مقترح** وليست تكاملات منفذة حاليًا.

## 1. المخطط التنفيذي الفاخر

```mermaid
%%{init: {"theme":"base", "themeVariables":{"primaryColor":"#0f766e","primaryTextColor":"#ffffff","primaryBorderColor":"#115e59","lineColor":"#64748b","secondaryColor":"#ecfeff","tertiaryColor":"#f8fafc","fontFamily":"Arial"}}}%%
flowchart LR
    classDef user fill:#f59e0b,stroke:#b45309,color:#fff,stroke-width:2px
    classDef client fill:#0f766e,stroke:#115e59,color:#fff,stroke-width:2px
    classDef core fill:#2563eb,stroke:#1d4ed8,color:#fff,stroke-width:2px
    classDef data fill:#7c3aed,stroke:#6d28d9,color:#fff,stroke-width:2px
    classDef cloud fill:#0891b2,stroke:#0e7490,color:#fff,stroke-width:2px
    classDef ops fill:#475569,stroke:#334155,color:#fff,stroke-width:2px

    U[المستخدمون]:::user --> M[تطبيق Android<br/>Kotlin + Jetpack Compose]:::client

    subgraph DEVICE[طبقة الجهاز]
        M --> NAV[Navigation Compose]:::client
        M --> UX[الشاشات والمكونات<br/>بحث • ملفات • التقرير الدستوري]:::client
        UX --> VM[ViewModel<br/>إدارة الحالة]:::core
        VM --> REPO[Repository<br/>مصدر البيانات الموحد]:::core
        REPO --> LOCAL[(Room Database<br/>وضع عدم الاتصال)]:::data
    end

    REPO --> API[بوابة API آمنة]:::cloud
    API --> AUTH[المصادقة وإدارة الجلسات]:::cloud
    API --> FILES[خدمة الملفات والمستندات]:::cloud
    API --> SYNC[المزامنة والإشعارات]:::cloud
    FILES --> OBJECT[(Object Storage)]:::data
    API --> DB[(Cloud Database)]:::data

    API --> OBS[مراقبة • سجلات • تنبيهات]:::ops
    OBJECT --> BACKUP[نسخ احتياطي واستعادة]:::ops
    DB --> BACKUP
```

## 2. مخطط AWS المقترح

```mermaid
flowchart LR
    App[تطبيق Android] --> Edge[Amazon CloudFront + AWS WAF]
    Edge --> API[Amazon API Gateway]
    API --> Auth[Amazon Cognito]
    API --> Lambda[AWS Lambda / ECS Fargate]
    Lambda --> DB[(Amazon Aurora / DynamoDB)]
    Lambda --> S3[(Amazon S3<br/>المستندات والنسخ)]
    Lambda --> Queue[Amazon SQS]
    Queue --> Worker[Lambda Workers]
    App -. إشعارات .-> SNS[Amazon SNS / FCM]
    Lambda --> Logs[Amazon CloudWatch]
    DB --> Backup[AWS Backup]
    S3 --> Backup
```

## 3. مخطط Microsoft Azure المقترح

```mermaid
flowchart LR
    App[تطبيق Android] --> Front[Azure Front Door + WAF]
    Front --> API[Azure API Management]
    API --> Identity[Microsoft Entra ID / B2C]
    API --> Compute[Azure Functions / App Service]
    Compute --> DB[(Azure SQL / Cosmos DB)]
    Compute --> Blob[(Azure Blob Storage<br/>المستندات والنسخ)]
    Compute --> Bus[Azure Service Bus]
    Bus --> Worker[Azure Functions Workers]
    App -. إشعارات .-> Notify[Azure Notification Hubs / FCM]
    Compute --> Monitor[Azure Monitor + Application Insights]
    DB --> Backup[Azure Backup]
    Blob --> Backup
```

## 4. مخطط Google Cloud المقترح

```mermaid
flowchart LR
    App[تطبيق Android] --> CDN[Cloud CDN + Cloud Armor]
    CDN --> Gateway[API Gateway]
    Gateway --> Auth[Firebase Authentication / Identity Platform]
    Gateway --> Compute[Cloud Run / Cloud Functions]
    Compute --> DB[(Firestore / Cloud SQL)]
    Compute --> Storage[(Cloud Storage<br/>المستندات والنسخ)]
    Compute --> PubSub[Pub/Sub]
    PubSub --> Worker[Cloud Run Jobs / Functions]
    App -. إشعارات .-> FCM[Firebase Cloud Messaging]
    Compute --> Observe[Cloud Logging + Monitoring]
    DB --> Backup[Backup and DR]
    Storage --> Backup
```

## 5. مقارنة سريعة للخيارات

| المجال | AWS | Azure | Google Cloud |
|---|---|---|---|
| الهوية | Cognito | Entra ID / B2C | Firebase Auth / Identity Platform |
| واجهة API | API Gateway | API Management | API Gateway |
| الحوسبة | Lambda أو ECS | Functions أو App Service | Cloud Run أو Functions |
| قاعدة البيانات | Aurora أو DynamoDB | Azure SQL أو Cosmos DB | Cloud SQL أو Firestore |
| تخزين الملفات | S3 | Blob Storage | Cloud Storage |
| الرسائل | SQS / SNS | Service Bus / Notification Hubs | Pub/Sub / FCM |
| المراقبة | CloudWatch | Azure Monitor | Cloud Monitoring |

## 6. التوصية

- **لأسرع إطلاق لتطبيق Android:** Google Cloud مع Firebase Authentication وFirestore وCloud Storage وFCM.
- **لبيئة مؤسسية وهوية Microsoft:** Azure مع Entra ID وApp Service وAzure SQL.
- **لبنية واسعة وقابلة للتوسع:** AWS مع Cognito وAPI Gateway وLambda وS3 وAurora/DynamoDB.
- يُفضّل إبقاء `DriveRepository` كطبقة عزل، بحيث يمكن تبديل مزود السحابة دون إعادة بناء واجهة التطبيق.

## تدفق البيانات السحابي

1. يسجل المستخدم الدخول من تطبيق Android.
2. تتحقق خدمة الهوية من الجلسة وتصدر رمز وصول.
3. يرسل التطبيق الطلب إلى بوابة API عبر HTTPS.
4. تتحقق الخدمة من الصلاحيات وتقرأ أو تكتب البيانات.
5. تُخزّن الملفات الكبيرة في Object Storage، بينما تُخزّن البيانات الوصفية في قاعدة البيانات.
6. تُرسل الأعمال غير المتزامنة إلى Queue أو Pub/Sub لمعالجتها بواسطة Workers.
7. تُحفظ السجلات والتنبيهات والنسخ الاحتياطية عبر خدمات التشغيل السحابي.
