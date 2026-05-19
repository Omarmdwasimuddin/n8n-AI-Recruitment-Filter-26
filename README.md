# 🧠 n8n AI Recruitment Filter
### AI-চালিত CV স্ক্রিনিং ও অটোমেটিক ইমেইল ওয়ার্কফ্লো

---

## 📋 ওয়ার্কফ্লো ওভারভিউ

```
Webhook → Extract from PDF → OpenAI (CV Analysis) → Supabase (Store) → IF (Score ≥ 70%) → Gmail
```

---

## 🔷 ধাপ ১ — Webhook নোড সেটআপ

> **কাজ:** ব্রাউজার থেকে PDF রিসিভ করা

### পদক্ষেপ:
1. **Nodes Panel** খুলুন
2. সার্চ করুন → **`Webhook`** ক্লিক করুন
3. নিচের মতো কনফিগার করুন:

| ফিল্ড | মান |
|---|---|
| HTTP Method | `POST` |
| Path | `talent-ai` |

4. **Test URL** কপি করুন
5. `index.html` ফাইলে URL **paste** করুন
6. Webhook নোডে **"Listen for Test Event"** ক্লিক করুন
7. ব্রাউজারে `index.html` ওপেন করুন
8. **PDF ফাইল আপলোড** করুন (Resume)

---

## 🔷 ধাপ ২ — Extract from File নোড সেটআপ

> **কাজ:** আপলোড করা PDF থেকে টেক্সট বের করা

### পদক্ষেপ:
1. Webhook নোডের **`+`** আইকনে ক্লিক করুন
2. সার্চ করুন → **`Extract from File`** ক্লিক করুন
3. নিচের মতো কনফিগার করুন:

| ফিল্ড | মান |
|---|---|
| Operation | `Extract From PDF` |
| Input Binary Field | `resume` |

4. **"Execute Step"** ক্লিক করুন ✅

---

## 🔷 ধাপ ৩ — OpenAI নোড সেটআপ

> **কাজ:** CV বিশ্লেষণ করে JSON স্কোর ও তথ্য বের করা

### পদক্ষেপ:
1. Extract from File নোডের **`+`** আইকনে ক্লিক করুন
2. সার্চ করুন → **`OpenAI`** ক্লিক করুন
3. **"Message a Model"** সিলেক্ট করুন
4. নিচের মতো কনফিগার করুন:

| ফিল্ড | মান |
|---|---|
| Model | *(পছন্দমতো, যেমন `gpt-4o`)* |
| Role | `System` |
| Prompt | *(Job description ও CV বিশ্লেষণের prompt দিন)* |

5. **"Add Option"** ক্লিক করুন:

| ফিল্ড | মান |
|---|---|
| Output Format | `JSON Object` |

6. **"Execute Step"** ক্লিক করুন ✅

> 💡 **Prompt উদাহরণ:**
> ```
> You are an expert HR recruiter. Analyze the provided CV against the job requirements.
> Return a JSON with: name, email, match_score (0-100), skills, experience_years, summary.
> ```

---

## 🔷 ধাপ ৪ — Supabase টেবিল তৈরি

> **কাজ:** CV ডেটা সংরক্ষণের জন্য ডেটাবেজ টেবিল বানানো

### Supabase-এ টেবিল তৈরি:
1. Supabase Dashboard খুলুন
2. **"Table Editor"** → **"New Table"** ক্লিক করুন
3. টেবিলের নাম দিন: **`CVs`**
4. নিচের কলামগুলো যোগ করুন:

| Column Name | Type | Extra |
|---|---|---|
| `id` | `int8` | Primary Key, Auto Increment |
| `name` | `text` | — |
| `email` | `text` | — |
| `match_score` | `int4` | — |
| `skills` | `text` | — |
| `experience_years` | `int4` | — |
| `summary` | `text` | — |
| `created_at` | `timestamptz` | Default: `now()` |


---

## 🔷 ধাপ ৫ — Supabase নোড সেটআপ

> **কাজ:** OpenAI-এর বিশ্লেষণ Supabase-এ সেভ করা

### পদক্ষেপ:
1. OpenAI নোডের **`+`** আইকনে ক্লিক করুন
2. সার্চ করুন → **`Supabase`** ক্লিক করুন
3. **"Create a Row"** সিলেক্ট করুন
4. নিচের মতো ম্যাপ করুন:

| Supabase Column | n8n Value |
|---|---|
| `name` | OpenAI output → `name` |
| `email` | OpenAI output → `email` |
| `match_score` | OpenAI output → `match_score` |
| `skills` | OpenAI output → `skills` |
| `experience_years` | OpenAI output → `experience_years` |
| `summary` | OpenAI output → `summary` |

5. **"Execute Step"** ক্লিক করুন ✅

---

## 🔷 ধাপ ৬ — IF নোড সেটআপ

> **কাজ:** Match Score ৭০% বা তার বেশি হলে ইমেইল পাঠানো

### পদক্ষেপ:
1. Supabase নোডের **`+`** আইকনে ক্লিক করুন
2. সার্চ করুন → **`IF`** ক্লিক করুন
3. **Condition** সেট করুন:

| ফিল্ড | মান |
|---|---|
| Value 1 | Supabase/OpenAI output → `match_score` |
| Operator | `≥ (Greater than or equal to)` |
| Value 2 | `70` |

---

## 🔷 ধাপ ৭ — Gmail নোড সেটআপ

> **কাজ:** যোগ্য প্রার্থীকে অটোমেটিক ইমেইল পাঠানো

### পদক্ষেপ:
1. IF নোডের **`True`** আউটপুটের **`+`** আইকনে ক্লিক করুন
2. সার্চ করুন → **`Gmail`** ক্লিক করুন
3. **"Send a Message"** সিলেক্ট করুন
4. নিচের মতো কনফিগার করুন:

| ফিল্ড | মান |
|---|---|
| To | OpenAI output → `email` |
| Subject | `Congratulations! Interview Invitation` |
| Body | প্রার্থীর নাম ও পরবর্তী পদক্ষেপ সম্পর্কে মেসেজ |

> 💡 **Body উদাহরণ:**
> ```
> Dear {{name}},
>
> We reviewed your resume and are impressed with your profile.
> We'd love to invite you for an interview.
>
> Best regards,
> HR Team
> ```

---

## 🗺️ সম্পূর্ণ ফ্লো চার্ট

```
┌──────────────────┐
│     Webhook      │  ◄── index.html থেকে PDF আপলোড
│  POST /talent-ai │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Extract from PDF │  ◄── resume ফিল্ড থেকে টেক্সট বের করা
│  (Input: resume) │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│     OpenAI       │  ◄── CV বিশ্লেষণ → JSON Output
│  (JSON Object)   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    Supabase      │  ◄── CVs টেবিলে ডেটা সেভ
│  (Create a Row)  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│       IF         │  ◄── match_score ≥ 70?
│  (Score Check)   │
└────────┬─────────┘
         │
    ┌────┴────┐
    │         │
  TRUE      FALSE
    │         │
    ▼         ▼
┌────────┐  ┌──────────────┐
│ Gmail  │  │  (শেষ হয়ে  │
│(Email) │  │   যায়)      │
└────────┘  └──────────────┘
```

---

## ✅ কুইক চেকলিস্ট

- [ ] Webhook নোড: Method = `POST`, Path = `talent-ai`
- [ ] `index.html` এ Test URL পেস্ট করা হয়েছে
- [ ] Extract from File: Operation = `Extract From PDF`, Field = `resume`
- [ ] OpenAI: Model, System Prompt ও Output Format = `JSON Object` সেট করা হয়েছে
- [ ] Supabase-এ **`CVs`** টেবিল তৈরি করা হয়েছে (সব কলামসহ)
- [ ] Supabase নোডে সব ফিল্ড ম্যাপ করা হয়েছে
- [ ] IF নোডে `match_score ≥ 70` কন্ডিশন সেট করা হয়েছে
- [ ] Gmail নোড: To, Subject, Body সেট করা হয়েছে (True branch)
- [ ] ওয়ার্কফ্লো **Activate** করা হয়েছে 🚀

---

> 💡 **টিপস:** প্রতিটি নোড সেটআপের পর **"Execute Step"** দিয়ে আউটপুট চেক করুন। সব ঠিকঠাক হলে উপরে **"Active"** টগল চালু করুন।
