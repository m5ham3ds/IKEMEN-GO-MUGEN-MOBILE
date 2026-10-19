# IKEMEN-GO Source and Workflow Audit (Arabic)

## النتيجة التنفيذية

أُعدّت نسخة العمل من أرشيف `Ikemen-GO-develop.zip` لأنه أحدث بوضوح من أرشيف `Ikemen-GO-release-1.0.zip` في أدوات البناء واعتماديات Go. لم تُدمج ملفات المصدر من الفرعين واحدًا بواحد؛ فذلك قد يتراجع عن إصلاحات `develop`. حُفظت ملفات المحرك وموارده كما وردت من أرشيف `develop`، وأُجري التنظيف في طبقة أتمتة GitHub والتوجيهات فقط.

**تنبيه للمصدر:** كلا الأرشيفين المضغوطين لا يحتويان تاريخ Git؛ لذلك لا يمكن من محتوى ZIP وحده إثبات commit upstream دقيق. التوصية تختار نسخة `develop` المرفوعة فعليًا، لا اعتمادًا على تخمين commit.

## مقارنة النسختين

| العنصر | أرشيف `develop` | أرشيف `release/1.0` | القرار |
|---|---|---|---|
| `go.mod` | `go 1.27.0` | `go 1.20` | استخدام develop لبناء حديث |
| Android Dockerfile | Ubuntu 24.04، Go 1.27.1، NDK r27d، SDK platform 34، build-tools 34.0.0 | Ubuntu 22.04، Go 1.20.14، NDK r27d، SDK platform 34، build-tools 34.0.0 | الاحتفاظ بإعداد develop الرسمي |
| Android build scripts | `build/build_android.sh` و`build/docker/android/*` موجودة | موجودة أيضًا لكن محتواها ليس مطابقًا بالكامل | الاحتفاظ بنسخة develop كاملة |
| ملفات مصدر Go/الموارد | أكثر تحديثًا، مع ملفات إضافية وتغييرات كثيرة | أقدم | عدم تركيب ملفات المصدر من الفرعين انتقائيًا |
| `.github/workflows` | 8 workflows | 8 workflows، مع اختلافات في `releases.yml` و`lint-code-style.yml` | تحليل السلوك وحذفها من المسار النشط ثم إضافة build workflow خاص بنا |
| `.github/dependabot.yml` | تحديث GitHub Actions وGo modules أسبوعيًا، مع سياسة تجاهل لبعض اعتمادات المشروع | تحديث GitHub Actions فقط | الاحتفاظ بسياسة develop لأنها تدعم اقتراح تحديثات الاعتماديات؛ لا دمج تلقائي |

## مراجعة كل Workflow موروث من نسخة develop

| الملف | ماذا يفعل فعليًا | القرار في النسخة المنظفة | السبب |
|---|---|---|---|
| `conventional-label.yml` | عند `pull_request_target` يضيف Labels حسب نوع PR ويحتاج كتابة labels | إزالته من workflows النشطة | ليس مطلوبًا للبناء، ويضيف أتمتة PR خاصة بأسلوب upstream |
| `conventional-title.yml` | يتحقق من عنوان PR، ويعلّق/يطلب تغييرات | إزالته | غير مطلوب لبناء المكتبات |
| `lint-code-style.yml` | يعمل عند push إلى فروع upstream أو يدويًا، ويستخدم `wearerequired/lint-action` مع `auto_fix: true` و`contents: write` ويستخدم `IKEMEN_TOKEN` | إزالته كما هو | قد يعدّل/يلتزم بالشيفرة ويحتاج سرًا وصلاحيات كتابة؛ يمكن إعادة بناء فحص آمن لاحقًا دون auto-fix |
| `releases.yml` | يبني Windows/Linux/macOS/Android/iOS، يرفع Artifacts، يحسب Checksums، يغيّر Tags المتحركة، وينشئ/يحدّث GitHub Releases | إزالته من workflows النشطة | غير مطلوب لبناء `.so` فقط، ويحتاج `contents: write` و`IKEMEN_TOKEN` وسلوك نشر واسع؛ لا نفعّله قبل إعداد نظام نشر خاص بنا |
| `repro-needed-stale.yml` | يعمل مجدولًا كل 12 ساعة أو يدويًا ويعلّم ويغلق Issues | إزالته | لا يساهم في بناء المحرك وقد يغلق Issues آليًا |
| `update-licence.yml` | يحدّث قائمة المساهمين ويعمل commit/push إلى `develop` مع `IKEMEN_TOKEN` | إزالته | يرتبط بفرع upstream وسرّ write token؛ ليس مطلوبًا للتطوير الحالي |
| `update-script-wiki.yml` | يولّد صفحة Lua في Wiki ثم يدفع التعديل؛ يستخدم `WIKI_PUSH_TOKEN` أو Token بديلًا | إزالته | لا نحتاج تحديث Wiki تلقائيًا الآن ويحتمل الكتابة إلى المستودع/Wiki |
| `wiki-docs.yml` | ينفذ gate للـ PR ويكتب تعليقات وفق Label `docs-needed` | إزالته | سياسة مراجعة خاصة بالمشروع الأصلي وليست عائقًا في البناء |

## مراجعة سكربتات `.github/scripts/`

| السكربت | المستخدم الأصلي | القرار |
|---|---|---|
| `generate_altstore_source.py` | Workflow النشر، لإنشاء metadata لتطبيق iOS IPA | أُزيل؛ ليس مطلوبًا لبناء Android `.so` |
| `generate_release_notes.py` | `releases.yml` لإنشاء ملاحظات إصدار مع دعم cherry-pick | أُزيل من النسخة النظيفة؛ يُسترجع فقط إذا صُمم نشر tagged releases لاحقًا |
| `generate_script_wiki.py` | `update-script-wiki.yml` | أُزيل مع Workflow الـ Wiki |
| `update_licence_contributors.py` | `update-licence.yml` | أُزيل مع Workflow تحديث المساهمين |

## الملفات التي ينبغي الاحتفاظ بها

- `src/` بكل ملفات Go المضمنة الخاصة بالمنصات؛ لا تُحذف ملفات iOS/Linux/Windows تلقائيًا، فقد تكون شروط build tags أو ملفات دعم ضرورية.
- `data/`, `font/`, `external/`, ملفات shaders والموارد المضمّنة.
- `go.mod`, `go.sum`, `Makefile`, `build/build.sh`, `build/build_android.sh`, `build/docker/android/Dockerfile`, `build/docker/android/docker-compose.yml`, `.gitignore`, `.gitattributes`, `LICENCE.txt`.
- `README.md` و`BUILDING.md` والوثائق الأصلية؛ أضيفت ملاحظة عن كون هذه نسخة عمل MUGEN-MOBILE، ولم تُستبدل تعليمات upstream التقنية.
- `.github/dependabot.yml` للاقتراحات الأسبوعية لتحديث Go modules وGitHub Actions؛ مراجعة كل PR يدويًا قبل الدمج.
- `.github/release.yml` كإعداد سلبي لتجميع ملاحظات الإصدارات مستقبلًا؛ لا يشغّل عملًا بحد ذاته.

## ما أُضيف

- `.github/workflows/android-native-build.yml`: يدوي فقط (`workflow_dispatch`)، `contents: read`، دون Secrets، ودون تشغيل تلقائي على push، ودون نشر إصدار. يبني `.so` باستخدام بناء Docker الرسمي مع `BUILD_ANDROID_APK=0`، ويتحقق من `AArch64`، ويرفع المكتبات و`libmain.h` و`BUILD_INFO.txt` و`SHA256SUMS.txt` كـ Artifact لمدة 14 يومًا.
- `MUGEN_MOBILE_ENGINE_WORKFLOW_GUIDE.md`: يشرح ما أزيل، وكيف يعمل بناء المكتبات وكيف كانت آلية النشر الأصلية، ومتطلبات أي Workflow نشر مستقبلي.

## ملاحظة تشغيل GitHub Actions

GitHub يتطلب أن يوجد Workflow الذي يستخدم `workflow_dispatch` في الفرع الافتراضي حتى يظهر زر التشغيل اليدوي في واجهة Actions. عند رفع النسخة النظيفة، يجب وضع الملف الجديد على الفرع الافتراضي أو تغيير الفرع الافتراضي بعد مراجعة المحتويات. لم يُشغّل بناء فعلي هنا ولم تُختبر المكتبات على جهاز Android بعد.

## خطة نشر مستقبلية آمنة

لا ينبغي تفعيل `releases.yml` الموروث كما هو. يبيّن الملف الأصلي أسلوب المطورين: إعداد tag/version، بناء مصفوفة المنصات، رفع artifacts، تجهيز Checksums وملاحظات الإصدار، ثم استدعاء `ncipollo/release-action` باستخدام `IKEMEN_TOKEN`. لكن ذلك مرتبط بفروعهم ونظام إصداراتهم. إذا أردنا نشر إصدارنا لاحقًا، ننشئ workflow منفصلًا يبدأ بـ `workflow_dispatch`، وننشر كـ Draft أولًا، ونمنح `contents: write` فقط عند الحاجة، ونحدد ملفات المكتبات أو APK التي تمت مراجعتها بالضبط. لا نحتاج لأي Token خاص لبناء `.so` ورفعها كـ Artifact.
