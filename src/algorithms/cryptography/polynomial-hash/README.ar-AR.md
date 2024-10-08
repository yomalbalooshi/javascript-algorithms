# التجزئة الدوارة متعددة الحدود

## دالة التجزئة

تُستخدم **دوال التجزئة** لتعيين مجموعات بيانات كبيرة من العناصر ذات طول عشوائي (_المفاتيح_) إلى مجموعات بيانات أصغر من العناصر ذات طول ثابت (_البصمات_).

التطبيق الأساسي للتجزئة هو الاختبار الفعال لتساوي المفاتيح من خلال مقارنة بصماتها.

يحدث _التصادم_ عندما يكون لمفتاحين مختلفين نفس البصمة. الطريقة التي يتم بها التعامل مع التصادمات حاسمة في معظم تطبيقات التجزئة. التجزئة مفيدة بشكل خاص في بناء خوارزميات عملية فعالة.

## التجزئة الدوارة

**التجزئة الدوارة** (تُعرف أيضًا باسم التجزئة التكرارية أو المجموع الاختباري الدوار) هي دالة تجزئة حيث يتم تجزئة المدخلات في نافذة تتحرك عبر المدخلات.

تسمح بعض دوال التجزئة بحساب التجزئة الدوارة بسرعة كبيرة - يتم حساب قيمة التجزئة الجديدة بسرعة بناءً على البيانات التالية فقط:

- قيمة التجزئة القديمة،
- القيمة القديمة المزالة من النافذة،
- والقيمة الجديدة المضافة إلى النافذة.

## تجزئة السلاسل متعددة الحدود

يجب أن تعتمد دالة التجزئة المثالية للسلاسل بشكل واضح على كل من _مجموعة_ الرموز الموجودة في المفتاح وعلى _ترتيب_ الرموز. العائلة الأكثر شيوعًا لمثل هذه الدوال التجزئة تعامل رموز السلسلة كمعاملات _متعددة حدود_ مع متغير صحيح `p` وتحسب قيمته بالنسبة لثابت صحيح `M`:

غالبًا ما يتم شرح _خوارزمية البحث عن السلاسل Rabin-Karp_ باستخدام دالة تجزئة دوارة بسيطة جدًا تستخدم فقط عمليات الضرب والإضافة - **التجزئة الدوارة متعددة الحدود**:

> H(s<sub>0</sub>, s<sub>1</sub>, ..., s<sub>k</sub>) = s<sub>0</sub> _ p<sup>k-1</sup> + s<sub>1</sub> _ p<sup>k-2</sup> + ... + s<sub>k</sub> \* p<sup>0</sup>

حيث `p` هو ثابت، و _(s<sub>1</sub>، ... ، s<sub>k</sub>)_ هي أحرف الإدخال.

على سبيل المثال، يمكننا تحويل السلاسل القصيرة إلى أرقام مفاتيح عن طريق ضرب رموز الأرقام في قوى ثابتة. يمكن تحويل الكلمة المكونة من ثلاثة أحرف `ace` إلى رقم عن طريق حساب:

> key = 1 _ 26<sup>2</sup> + 3 _ 26<sup>1</sup> + 5 \* 26<sup>0</sup>

لتجنب التعامل مع قيم `H` ضخمة، يتم إجراء جميع العمليات الحسابية بالنسبة لـ `M`.

> H(s<sub>0</sub>, s<sub>1</sub>, ..., s<sub>k</sub>) = (s<sub>0</sub> _ p<sup>k-1</sup> + s<sub>1</sub> _ p<sup>k-2</sup> + ... + s<sub>k</sub> \* p<sup>0</sup>) mod M

الاختيار الدقيق للمعلمات `M`، `p` مهم للحصول على خصائص "جيدة" لدالة التجزئة، أي معدل تصادم منخفض.

هذا النهج له السمة المرغوبة المتمثلة في إشراك جميع الأحرف في سلسلة الإدخال. يمكن بعد ذلك تجزئة قيمة المفتاح المحسوبة إلى فهرس مصفوفة بالطريقة المعتادة:

```javascript
function hash(key, arraySize) {
  const base = 13

  let hash = 0
  for (let charIndex = 0; charIndex < key.length; charIndex += 1) {
    const charCode = key.charCodeAt(charIndex)
    hash += charCode * base ** (key.length - charIndex - 1)
  }

  return hash % arraySize
}
```

طريقة `hash()` ليست فعالة كما يمكن أن تكون. بخلاف تحويل الأحرف، هناك عمليتا ضرب وإضافة داخل الحلقة. يمكننا إزالة عملية ضرب واحدة باستخدام **طريقة هورنر**:

> a<sub>4</sub> _ x<sup>4</sup> + a<sub>3</sub> _ x<sup>3</sup> + a<sub>2</sub> _ x<sup>2</sup> + a<sub>1</sub> _ x<sup>1</sup> + a<sub>0</sub> = (((a<sub>4</sub> _ x + a<sub>3</sub>) _ x + a<sub>2</sub>) _ x + a<sub>1</sub>) _ x + a<sub>0</sub>

بعبارة أخرى:

> H<sub>i</sub> = (P \* H<sub>i-1</sub> + S<sub>i</sub>) mod M

لا يمكن لـ `hash()` التعامل مع السلاسل الطويلة لأن hashVal تتجاوز حجم int. لاحظ أن المفتاح ينتهي دائمًا بأن يكون أقل من حجم المصفوفة. في طريقة هورنر يمكننا تطبيق عامل التشغيل modulo (٪) في كل خطوة من الحساب. هذا يعطي نفس النتيجة كتطبيق عامل التشغيل modulo مرة واحدة في النهاية، ولكنه يتجنب التجاوز.

```javascript
function hash(key, arraySize) {
  const base = 13

  let hash = 0
  for (let charIndex = 0; charIndex < key.length; charIndex += 1) {
    const charCode = key.charCodeAt(charIndex)
    hash = (hash * base + charCode) % arraySize
  }

  return hash
}
```

التجزئة متعددة الحدود لها خاصية دوارة: يمكن تحديث البصمات بكفاءة عند إضافة الرموز أو إزالتها من نهايات السلسلة (بشرط تخزين مصفوفة من قوى p بالنسبة لـ M بطول كافٍ). تعتمد خوارزمية مطابقة النمط Rabin-Karp الشائعة على هذه الخاصية

## المراجع

- [أين تُستخدم دالة التجزئة للسلاسل النصية متعددة الحدود](https://www.mii.lt/olympiads_in_informatics/pdf/INFOL119.pdf)
- [التجزئة على موقع جامعة تكساس](https://www.cs.utexas.edu/~mitra/csSpring2017/cs313/lectures/hash.html)
- [دالة التجزئة على ويكيبيديا](https://en.wikipedia.org/wiki/Hash_function)
- [التجزئة المتدحرجة على ويكيبيديا](https://en.wikipedia.org/wiki/Rolling_hash)
