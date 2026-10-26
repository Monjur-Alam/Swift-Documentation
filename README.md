# Swift Documentation

![image](https://user-images.githubusercontent.com/40492085/224463912-b3352fab-c1ad-4a09-8a7e-08c13d9dbf20.png)

আগের দুটি টপিক ইন্টারভিউ নোট হিসেবে সাজিয়ে দিলাম। ব্যাখ্যা বাংলায়, প্রশ্ন ও বলার মতো উত্তর ইংরেজিতে — লিখিত আর ওরাল দুটোতেই কাজে লাগবে। পরের টপিকগুলোও এই ফরম্যাটে দেব।

---

# 1. Variables ও Constants (`var` / `let`)

## ৩০ সেকেন্ডে বলার উত্তর

`var` creates a mutable binding. `let` creates a binding that can be assigned exactly once. `let` does not mean deep immutability: a `let` class instance can still change its `var` properties, because only the reference is fixed. A `let` struct cannot change any stored property. Prefer `let` unless mutation is required. Global and static stored constants are lazy and thread-safe. A local `lazy var` is not thread-safe.

## যে পয়েন্টগুলো বলতে হবে

**Binding আর value আলাদা জিনিস।** `let` মানে এই নামে আর নতুন মান বসানো যাবে না। ভিতরের অবজেক্ট অপরিবর্তনীয়, এমন কথা `let` একা বলে না।

```swift
struct User { var name: String }
class Account { var balance: Int; init(balance: Int) { self.balance = balance } }

let user = User(name: "Ted")
user.name = "Roy"                 // compile error — struct value পরিবর্তন

let account = Account(balance: 10)
account.balance = 20              // বৈধ — reference একই, property var
account = Account(balance: 30)    // compile error — binding বদলানো যাচ্ছে
```

**Struct-এ mutation মানে পুরো value বদলানো।** `var` প্রপার্টি বদলাতে `mutating` মেথড লাগে। `let` ইনস্ট্যান্সে সেই মেথড কল হয় না।

```swift
struct Score {
    var value: Int
    mutating func add(_ n: Int) { value += n }
}

var s = Score(value: 1)
s.add(2)          // বৈধ

let fixed = Score(value: 1)
fixed.add(2)      // compile error
```

**`let` একবার অ্যাসাইন হয়, ডিক্লারেশনের লাইনেই হতে হবে এমন বাধ্যবাধকতা নেই।** সব পাথে ঠিক একবার সেট হতে হবে, ব্যবহারের আগে।

```swift
let score: Int
if passed {
    score = 100
} else {
    score = 0
}
```

**কোথায় lazy, কোথায় নয়।**

| জায়গা | আচরণ |
|---|---|
| Local `let` / `var` | লাইন এক্সিকিউট হলেই তৈরি হয় |
| Global `let` / `var` | প্রথম ব্যবহারের সময়, thread-safe |
| `static let` / `static var` | প্রথম ব্যবহারের সময়, thread-safe |
| `lazy var` | প্রথম ব্যবহারের সময়, **thread-safe নয়** |

`lazy` শুধু `var`-এর সাথে হয়। `let`-এর ডিফল্ট ভ্যালু ইনিশিয়ালাইজেশনেই সেট হয়।

**ক্লাসের `let` প্রপার্টি শুধু `init`-এ সেট করা যায়।** `init` শেষ হওয়ার পর সেই ইনস্ট্যান্স নিজেও সেটা বদলাতে পারে না।

**Value type-এর ভিতরে reference থাকলে কপি ভাগ হয়ে যায়।**

```swift
class Box { var n = 0 }
struct Holder { var box = Box() }

let a = Holder()
var b = a
b.box.n = 5
// a.box.n এখন 5 — Holder কপি হয়েছে, Box একই অবজেক্ট
```

Array, Dictionary, Set, String-এ copy-on-write আছে। সাধারণ struct-এ নেই।

**Closure capture।** ক্যাপচার লিস্ট না দিলে ক্লোজার ভেরিয়েবলের বর্তমান মান দেখে। `[name]` দিলে ক্লোজার তৈরির মুহূর্তের ভ্যালু কপি হয়। ক্লাস হলে রেফারেন্স কপি হয়, ডিপ কপি নয়।

```swift
var name = "Roy"
let snapshot = { [name] in name }
let live = { name }

name = "Sam"
snapshot()   // "Roy"
live()       // "Sam"
```

**SwiftUI।** `View` একটি struct। ইনপুট ডেটা `let` রাখা যায়। `@State` / `@StateObject` প্রপার্টি `var` হতে হয়, কারণ প্রপার্টি র‍্যাপার নিজের `wrappedValue` বদলায়। স্টেটের আসল স্টোরেজ ভিউ স্ট্রাকটের বাইরে থাকে।

```swift
struct Counter: View {
    let title: String
    @State private var count = 0

    var body: some View {
        Button(title) { count += 1 }
    }
}
```

**আরও ছোট নিয়ম, ইন্টারভিউতে উঠলে বলবেন।**

- ফাংশন প্যারামিটার ডিফল্টভাবে কনস্ট্যান্ট। Swift 3-এর পর `func f(var x: Int)` নেই। বদলাতে হলে ভিতরে `var copy = x` করুন। কলারের মান বদলাতে `inout`।
- `willSet` / `didSet` শুধু `var` স্টোরড প্রপার্টিতে।
- নামের নিয়ম camelCase। `playerName` আর `playername` আলাদা আইডেন্টিফায়ার।
- টাইপ ইনফারেন্স আছে। পাবলিক API-তে অনেক সময় এনোটেটেড টাইপ স্পষ্ট রাখা ভালো।

## MCQ

**1.** নিচের কোন লাইন কম্পাইল হবে?

```swift
class Person { var name: String; init(_ name: String) { self.name = name } }
let p = Person("Ted")
```

A. `p = Person("Roy")`  
B. `p.name = "Roy"`  
C. দুটোই  
D. কোনোটাই না  

**Answer: B.** `let` রেফারেন্স আটকায়। ক্লাসের `var` প্রপার্টি বদলানো যায়।

**2.** Global `let` সম্পর্কে কোনটা ঠিক?

A. প্রোগ্রাম স্টার্টেই তৈরি হয় এবং thread-safe নয়  
B. প্রথম অ্যাক্সেসে তৈরি হয় এবং thread-safe  
C. `lazy` না লিখলে তৈরিই হয় না  
D. প্রতিবার অ্যাক্সেসে নতুন করে তৈরি হয়  

**Answer: B.** Global ও `static` stored property lazily initialize হয়, `swift_once`-স্টাইলে।

**3.** `lazy var cache: Image` দুই থ্রেড থেকে একসাথে প্রথমবার পড়লে কী ঝুঁকি?

A. কোনো ঝুঁকি নেই, রানটাইম সিরিয়ালাইজ করে  
B. ইনিশিয়ালাইজার একাধিকবার চলতে পারে  
C. কম্পাইলার এটা `let`-এ বদলে দেয়  
D. অ্যাপ অবশ্যই কম্পাইল হবে না  

**Answer: B.** `lazy var` thread-safe নয়। শেয়ারড স্টেটে `static let` বা নিজের লক ব্যবহার করুন।

**4.** এই কোডে `a.n` কত?

```swift
class Box { var n = 1 }
struct Wrap { var box = Box() }
let a = Wrap()
var b = a
b.box.n = 9
```

A. 1  
B. 9  
C. কম্পাইল এরর  
D. রানটাইম ক্র্যাশ  

**Answer: B.** `Wrap` কপি হয়, ভিতরের `Box` একই রেফারেন্স।

**5.** কোনটা কম্পাইল হবে?

```swift
struct S { var n = 0; mutating func inc() { n += 1 } }
```

A. `let s = S(); s.inc()`  
B. `let s = S(); s.n = 1`  
C. `var s = S(); s.inc()`  
D. `func f(var x: Int) { x += 1 }`  

**Answer: C.** `mutating` কল করতে ইনস্ট্যান্স `var` হতে হবে। `var` প্যারামিটার সিনট্যাক্স Swift 3-এ সরানো হয়েছে।

**6.** `@State` প্রপার্টি `let` লেখা যায় না কেন?

A. SwiftUI ভিউ ক্লাস, ক্লাসে `let` নিষিধ  
B. প্রপার্টি র‍্যাপার `wrappedValue` মিউটেট করে, তাই `var` লাগে  
C. `let` মেইন থ্রেডে চলে না  
D. `@State` শুধু ক্লাসে কাজ করে  

**Answer: B.** `View` struct-ই। স্টোরেজ ভিউয়ের বাইরে, তবু র‍্যাপার প্রপার্টি `var`।

## Short answer

**প্রশ্ন ১. কখন `let` ব্যবহার করবেন, কখন `var`?**  
যে বাইন্ডিং ইচ্ছাকৃতভাবে বদলাবে না, সেখানে `let` ব্যবহার করুন। এতে উদ্দেশ্য স্পষ্ট থাকে, struct-এর মান ভুল করে বদলানো যায় না, এবং কম্পাইলার নিশ্চিত করে যে অ্যাসাইনমেন্ট একবারই হয়েছে। পারফরম্যান্সে লাভ সাধারণত সামান্য। আসল লাভ সঠিকতা। `var` তখনই ব্যবহার করুন, যখন মান সত্যিই বদলাতে হবে। যেমন `lazy var`, লুপ ভেরিয়েবল, এবং `@State`-এর মতো প্রপার্টি র‍্যাপার।

**প্রশ্ন ২. `let` কি অবজেক্টকে অপরিবর্তনীয় করে?**  
`let` বাইন্ডিংকে অপরিবর্তনীয় করে। struct হলে স্টোরড প্রপার্টি আর বদলায় না, তাই পুরো ভ্যালু স্থির থাকে। ক্লাস হলে রেফারেন্স স্থির থাকে, কিন্তু তার `var` প্রপার্টি বদলানো যায়। struct-এর ভিতরে ক্লাস থাকলে struct কপি হলেও সেই ক্লাসের রেফারেন্স শেয়ার হয়। পুরো অবজেক্ট গ্রাফ ভ্যালু টাইপ না হলে, বা ক্লাস নিজে মিউটেশন না দিলে, ইমিউটেবিলিটি গভীর হয় না।

**প্রশ্ন ৩. থ্রেডের মধ্যে শেয়ার করলে `lazy var` অনিরাপদ, আর `static let` নিরাপদ কেন?**  
`lazy var` সিঙ্ক্রোনাইজেশন ছাড়াই চেক করে ও মান বসায়। দুই থ্রেড একসাথে দেখতে পারে যে এখনো ইনিশিয়ালাইজ হয়নি, তারপর দুইজনই ইনিশিয়ালাইজার চালায়। গ্লোবাল ও `static` স্টোরড প্রপার্টি রানটাইমের `once` টোকেন দিয়ে একবার, লেইজিলি ইনিশিয়ালাইজ হয়। দামি শেয়ারড স্টেটের জন্য `static let` ব্যবহার করুন। সত্যিই `lazy var` লাগলে নিজের লক দিন।

**প্রশ্ন ৪. ক্যাপচার লিস্ট কী কপি করে?**  
`{ [name] in }` ক্লোজার তৈরির সময় `name` ইভ্যালুয়েট করে সেই কপি রেখে দেয়। `let` বা ভ্যালু টাইপ হলে ক্লোজার আলাদা একটি মান পায়। ক্লাস ইনস্ট্যান্স হলে রেফারেন্স কপি হয়, তাই পরে অবজেক্ট বদলালে ক্লোজার সেই বদল দেখে। `[weak self]` একই পদ্ধতিতে রিটেইন সাইকেল এড়ায়।

**প্রশ্ন ৫. `inout` আর `var` কীভাবে আলাদা?**  
সাধারণ প্যারামিটার একটি লোকাল কনস্ট্যান্ট। কপি এডিট করতে ভিতরে `var` বানাতে হয়। `inout` একটি মিউটেবল মেমরি লোকেশন পাঠায়। ফাংশন কলারের ভেরিয়েবলে লিখে দেয়। কলারকে `var` পাস করতে হয়, সামনে `&` দিয়ে। ক্লাসের জন্য `inout` মানে “অবজেক্ট রেফারেন্স হিসেবে পাঠানো” নয়। ক্লাস নিজেই রেফারেন্স। ক্লাস প্যারামিটারে `inout` দিলে কলারের ভেরিয়েবল কোন অবজেক্টকে ধরবে, সেটা বদলায়।

---

# 2. Strings

## ৩০ সেকেন্ডে বলার উত্তর

`String` is a value-type struct with copy-on-write and Unicode-correct `Character`s. A `Character` is an extended grapheme cluster, so emoji and combining marks often count as one character. Because clusters have variable width, `String.Index` is not an `Int`, and `count` walks the string, so it is O(n). `isEmpty` is O(1). `NSString.length` counts UTF-16 code units, which is why the same emoji can be 1 as `String` and 2 as `NSString`. Equality uses canonical equivalence. A `Substring` shares the original buffer. Do not keep it alive if that would pin a large string.

## যে পয়েন্টগুলো বলতে হবে

**তিনটা ভিউ মিশিয়ে ফেলবেন না।**

| API | কী গোনে |
|---|---|
| `string.count` | `Character` — extended grapheme cluster |
| `string.unicodeScalars.count` | Unicode scalar |
| `string.utf8.count` / `utf16.count` | কোড ইউনিট |
| `(string as NSString).length` | UTF-16 কোড ইউনিট |

```swift
let thumbs = "👍"
thumbs.count                    // 1
thumbs.unicodeScalars.count     // 1
thumbs.utf16.count              // 2
(thumbs as NSString).length     // 2

let flag = "🇺🇸"
flag.count                      // 1
flag.unicodeScalars.count       // 2
flag.utf16.count                // 4

let family = "👨‍👩‍👧‍👦"
family.count                    // 1
```

**`count` O(n), `isEmpty` O(1)।** প্রতিবার `count` কল করে লুপের শর্ত লিখলে লম্বা স্ট্রিংয়ে খরচ বাড়ে। খালি কি না, সেটা `isEmpty` দিয়ে দেখুন।

**ইন্ডেক্স ইন্টিজার নয়।** র‍্যান্ডম অ্যাক্সেস নেই, তাই `string[0]` কম্পাইল হয় না। `index(_:offsetBy:)` ও O(n)।

```swift
let name = "Denzel"
let i = name.index(name.startIndex, offsetBy: 1)
name[i]   // "e"

name.index(name.startIndex, offsetBy: 100, limitedBy: name.endIndex) // nil
```

`endIndex` শেষ ক্যারেক্টারের পরের জায়গা। সেখানে সাবস্ক্রিপ্ট করা যায় না। খালি স্ট্রিংয়ে `startIndex == endIndex`।

এক স্ট্রিংয়ের `String.Index` অন্য স্ট্রিংয়ে ব্যবহার করলে রানটাইমে ট্র্যাপ হয়। `Substring` তার base string-এর ইন্ডেক্স শেয়ার করে।

**সমানতার নিয়ম Unicode canonical equivalence।** দেখতে এক হলেই সমান, বাইট বাইট মিলতে হবে না।

```swift
let composed = "\u{00E9}"       // é
let combining = "e\u{0301}"     // e + acute
composed == combining           // true
composed.count                  // 1
combining.count                 // 1
combining.unicodeScalars.count  // 2
```

**Case sensitivity।** `hasPrefix`, `hasSuffix`, `==` — সব কেস-সেনসিটিভ। `"paris.jpg".hasSuffix(".JPG")` → `false`।

কেস উঠানামায় দৈর্ঘ্য বদলাতে পারে। লোকেলও বদলায়।

```swift
"straße".uppercased()                              // "STRASSE"
"i".uppercased()                                   // "I"
"i".uppercased(with: Locale(identifier: "tr_TR"))  // "İ"
```

UI-তে ইউজারের ভাষা ধরতে `localizedStandardContains` বা লোকেলসহ API। ফাইল এক্সটেনশন বা আইডেন্টিফায়ারে লোকেল-অ্যাওয়্যার কেস পরিবর্তন এড়িয়ে চলুন।

**`count`-এ বন্ধনী নেই, `uppercased()`-এ আছে।** `count` একটি `Collection` প্রপার্টি। `uppercased()` নতুন স্ট্রিং বানিয়ে দেয়। একই নিয়ম: `isEmpty` প্রপার্টি, `hasPrefix(_:)` মেথড।

**মাল্টিলাইন ও এস্কেপ।**

```swift
let quote = "He said \"Believe\""

let movie = """
    A day in
    the life of an
    Apple engineer
    """
// বন্ধ করার """ যতটুকু ইন্ডেন্ট, প্রতিটি লাইন থেকে ততটুকু কেটে যায়

let raw = #"C:\Images\paris.jpg and \"no escape\""#
```

**পারফরম্যান্স।** `String` copy-on-write। `var` স্ট্রিংয়ে `append` করলে বাফার ইউনিক থাকলে ইন-প্লেস বাড়ে। লুপে `text = text + piece` বারবার কপি তৈরি করতে পারে। অনেক টুকরো জোড়া লাগালে `append` বা `reserveCapacity` ব্যবহার করুন।

**`Substring` পুরো বাফার ধরে রাখে।**

```swift
let big = String(repeating: "a", count: 1_000_000)
let first = big.prefix(1)     // Substring, big-এর স্টোরেজ জীবিত
let detached = String(first)  // নিজের স্টোরেজ, big ছেড়ে দেওয়া যায়
```

API-র বাইরে `Substring` বেশিক্ষণ রাখবেন না। প্রপার্টি বা ডিকশনারি কী হিসেবে `String` দিন।

**`Character` ঠিক একটি grapheme cluster।** `"AB"` একটি `Character`-এ বসানো যায় না। `"👍"` যায়।

## MCQ

**1.** `"👍".count` আর `( "👍" as NSString).length` কত?

A. 1 এবং 1  
B. 1 এবং 2  
C. 2 এবং 2  
D. 2 এবং 1  

**Answer: B.** `count` একটি `Character`। `NSString.length` দুটি UTF-16 কোড ইউনিট।

**2.** লক্ষাধিক ক্যারেক্টারের স্ট্রিং খালি কি না, সবচেয়ে ভালো কোন চেক?

A. `s.count == 0`  
B. `s.isEmpty`  
C. `s.utf8.count == 0`  
D. `(s as NSString).length == 0`  

**Answer: B.** `isEmpty` O(1)। `count` পুরো স্ট্রিং হাঁটে, তাই O(n)।

**3.** `"\u{00E9}" == "e\u{0301}"` এর ফল কী?

A. `false`, কারণ scalar সংখ্যা আলাদা  
B. `true`, canonical equivalence  
C. কম্পাইল এরর  
D. `true` শুধু `localizedCompare`-এ  

**Answer: B.** Swift-এর `==` canonically equivalent স্ট্রিংকে সমান ধরে। দুটোর `count`ই 1।

**4.** কোনটা কম্পাইল হবে এবং নিরাপদ?

A. `let s = "Swift"; s[0]`  
B. `let s = "Swift"; s[s.startIndex]`  
C. `let a = "hi", b = "hi"; b[a.startIndex]`  
D. `let c: Character = "Swift"`  

**Answer: B.** সাবস্ক্রিপ্ট `String.Index` চায়। অন্য স্ট্রিংয়ের ইন্ডেক্স রানটাইমে ট্র্যাপ করে। `Character`-এ একটির বেশি grapheme বসানো যায় না।

**5.** `"straße".uppercased()` কী দেয়?

A. `"STRAßE"`, দৈর্ঘ্য একই  
B. `"STRASSE"`, দৈর্ঘ্য বাড়ে  
C. `"strasse"`  
D. কম্পাইল এরর, কারণ `ß` ASCII নয়  

**Answer: B.** Unicode case mapping সবসময় 1:1 নয়। দৈর্ঘ্য বদলাতে পারে।

**6.** বড় একটি লগ স্ট্রিং থেকে `prefix(10)` নিয়ে প্রপার্টিতে রেখে দিলে কী সমস্যা?

A. `Substring` র‍্যান্ডম অ্যাক্সেস সাপোর্ট করে না  
B. `Substring` আসল বাফার ধরে রাখে, পুরো লগ মেমরিতে থাকে  
C. `prefix` সবসময় কপি করে, তাই দ্বিগুণ মেমরি লাগেই  
D. দশ ক্যারেক্টারের বেশি হলে ক্র্যাশ  

**Answer: B.** শেয়ার করা স্টোরেজ ছাড়াতে `String(substring)` বানিয়ে নিন।

**7.** `"photo.jpg".hasSuffix(".JPG")` কেন `false`?

A. `hasSuffix` শুধু চার ক্যারেক্টার পর্যন্ত দেখে  
B. তুলনা case-sensitive  
C. এটা `NSString` API, Swift স্ট্রিংয়ে নেই  
D. ডট থাকলে `hasSuffix` কাজ করে না  

**Answer: B.** `hasPrefix` / `hasSuffix` / `==` কেস-সেনসিটিভ। ফাইল এক্সটেনশন আগে নর্মালাইজ করে নিন।

## Short answer

**প্রশ্ন ১. `String.count` কেন O(n), আর ইন্টিজার সাবস্ক্রিপ্ট কেন নেই?**  
`Character` হলো extended grapheme cluster। UTF-8-এ এর প্রস্থ স্থির নয়। পঞ্চম ক্যারেক্টার অনেক ভিন্ন বাইট অফসেটে শুরু হতে পারে, তাই স্ট্রিংকে শুরু থেকে হেঁটে আসতে হয়। `String` একটি `BidirectionalCollection`। এটা র‍্যান্ডম-অ্যাক্সেস নয়। অন্য স্ট্রিংয়ের ইন্ডেক্স অবৈধ, রানটাইমে ট্র্যাপ করে। এগোতে `index(after:)`, `index(_:offsetBy:)`, বা `index(_:offsetBy:limitedBy:)` ব্যবহার করুন।

**প্রশ্ন ২. `String.count` আর `NSString.length` কীভাবে আলাদা?**  
`count` আঁকা ক্যারেক্টারের সংখ্যা। তাই থাম্বস-আপ ইমোজির `count` ১। `NSString.length` ও `utf16.count` UTF-16 কোড ইউনিট গোনে, তাই সেই ইমোজি ২, আর একটি ফ্ল্যাগ ৪ হতে পারে। Foundation বা UIKit-এ ব্রিজ করার সময় `NSString`-এর `NSRange` আর Swift-এর `String.Index` মেশালে অফ-বাই-ওয়ান বাগ হয়।

**প্রশ্ন ৩. প্রতিবার অ্যাসাইনমেন্টে পুরো কপি না করে `String` কীভাবে ভ্যালু টাইপ থাকে?**  
অ্যাসাইনমেন্টে struct-এর হেডার কপি হয় এবং একই স্টোরেজ রিটেইন হয়। মিউটেশনের সময় ইউনিক কি না, সেটা চেক হয়। বাফারের মালিক একজন হলে `append` জায়গাতেই বাড়ায়। অন্য ভেরিয়েবল সেই বাফার ব্যবহার করলে আগে কপি হয়, তারপর মিউটেশন হয়। এটার নাম copy-on-write। `Array`, `Dictionary`, `Set`-ও একই মডেল ব্যবহার করে।

**প্রশ্ন ৪. `uppercased()` কখন দৈর্ঘ্য বদলায়, আর লোকেল কখন গুরুত্বপূর্ণ?**  
Unicode-এর কেস ফোল্ডিং সবসময় এক ক্যারেক্টার থেকে এক ক্যারেক্টার নয়। জার্মান `ß` বড় হাতের অক্ষরে `SS` হয়। তুর্কি ভাষায় ডটবিহীন `ı` আর ডটযুক্ত `İ` আলাদা, তাই `uppercased(with:)` ডিফল্ট লোকেলের সাথে মিলবে না। ইউজার যা দেখে, সেই সার্চ ও সর্টে লোকেল-অ্যাওয়্যার API ব্যবহার করুন। আইডেন্টিফায়ার, ফাইল এক্সটেনশন, টোকেনে লোকেল-নিরপেক্ষ লোয়ারকেস ব্যবহার করুন, যাতে ডিভাইসের ভাষা বদলালে ফল না বদলায়।

**প্রশ্ন ৫. স্টোরড প্রপার্টি `Substring` না রেখে `String` রাখবেন কেন?**  
`prefix`, `suffix`, বা রেঞ্জ দিয়ে কাটলে `Substring` আসে, যা আসল বাফার শেয়ার করে। সেই স্লাইস ধরে রাখলে পুরো আসল অ্যালোকেশন মেমরিতে থাকে। `String(sub)` আলাদা স্টোরেজ বানায়, তারপর বড় সোর্স ছেড়ে দেওয়া যায়। ফাংশনের ভিতরে স্লাইস করা ঠিক আছে। প্রপার্টিতে রাখা, API থেকে রিটার্ন করা, বা ডিকশনারি কী বানানোর আগে `String`-এ বদলে নিন।

**প্রশ্ন ৬. বড় স্ট্রিং দক্ষভাবে কীভাবে বানাবেন?**  
`var` নিন এবং `append` করুন। সাইজ জানা থাকলে `reserveCapacity` কল করুন। বারবার `text = text + chunk` করলে বাড়ন্ত বাফার প্রতি ধাপে কপি হতে পারে। টুকরোর সংখ্যা নির্দিষ্ট হলে ইন্টারপোলেশন (`"\(a) \(b)"`) যথেষ্ট। অনেক টুকরো হলে একটি মিউটেবল স্ট্রিংয়ে `append` করুন।

# 3. Whole Numbers (`Int`)

## ৩০ সেকেন্ডে বলার উত্তর

`Int` is Swift’s signed integer, a small value-type struct. On every current Apple platform it is 64-bit, but it is a distinct type from `Int64`. Decimal, hex, binary, and underscores in literals are only formatting. The compiler ignores `_`. Ordinary `+`, `-`, `*`, and `/` trap on overflow in debug and release. Wrapping uses `&+`. Integer division truncates toward zero. `%` is a remainder, so it can be negative. `isMultiple(of:)` is the safe check, including for `0` and `Int.min`. There is no implicit conversion between numeric types.

## যে পয়েন্টগুলো বলতে হবে

**বেসিক লেখা।** `let` বা `var`, নাম, তারপর পূর্ণসংখ্যা। ঋণাত্মক মানও `Int`।

```swift
let score = 10
var counter = 10
let reallyBig = 100_000_000
let sameNumber = 1_00__00___00____00   // একই মান, 100_000_000
```

`_` শুধু পড়ার সুবিধা। কম্পাইলার আন্ডারস্কোর ফেলে দেয়। একসাথে অনেকগুলো `_` দিলেও মান বদলায় না।

অন্য লিটারাল:

```swift
let mask = 0b0000_1111   // binary, 15
let hex = 0xFF_00_AA     // hex
```

**অঙ্ক আর compound assignment।** নতুন ভ্যালু বানাতে `+ - * /`। আগের ভেরিয়েবলেই ফেরত রাখতে `+= -= *= /= %=`। বাঁ পাশ `var` হতে হয়।

```swift
let lowerScore = score - 2
let higherScore = score + 10
let doubledScore = score * 2
let squaredScore = score * score
let halvedScore = score / 2      // দুটোই Int হলে ভাগফল পূর্ণসংখ্যা

counter += 5
counter *= 2
counter -= 10
counter /= 2
```

`9 / 2` হয় `4`। দশমিক চাইলে ভাগের আগে ভাসমান সংখ্যায় নিন: `Double(score) / 2`।

**`+=` অ্যাটমিক নয়।** দুই থ্রেড একই `var counter += 1` করলে আপডেট হারাতে পারে। শেয়ারড কাউন্টারে অ্যাক্টর, লক, বা Swift Atomics ব্যবহার করুন। `Int` নিজে `Sendable`। শেয়ার করা `var` সেন্টেবল নয়।

**Overflow ডিফল্টে ট্র্যাপ করে।** `Int.max + 1` ডিবাগ ও রিলিজ দুই জায়গাতেই `fatal error: arithmetic overflow`। C বা Objective-C-র মতো চুপচাপ র‍্যাপ হয় না।

```swift
let wrapped = Int.max &+ 1          // Int.min, ট্র্যাপ নেই
let (value, didOverflow) = Int.max.addingReportingOverflow(1)
// value == Int.min, didOverflow == true
```

`&+ &- &* &<< &>>` র‍্যাপিং অপারেটর। হ্যাশ বা বিট অঙ্কে দরকার হলে ব্যবহার করুন। ক্র্যাশ এড়াতে সাধারণ হিসাবে এগুলো ব্যবহার করবেন না।

`abs(Int.min)` এবং `-Int.min` ও ট্র্যাপ করে। `Int.min`-এর ধনাত্মক রূপ `Int`-এ ধরে না, কারণ `Int.max` তার চেয়ে ১ ছোট।

**ভাগ ও ভাগশেষ।** শূন্য দিয়ে ভাগ করলে ট্র্যাপ। `%` mathematical modulo নয়। এটা remainder। চিহ্ন বাঁ পাশের সংখ্যাটিকে অনুসরণ করে। ভাগফল শূন্যের দিকে কাটে।

```swift
9 / 2      // 4
-9 / 2     // -4
-9 % 2     // -1
9 % -2     // 1
```

ধনাত্মক modulo দরকার হলে:

```swift
func mod(_ a: Int, _ n: Int) -> Int {
    let r = a % n
    return r >= 0 ? r : r + n
}
```

`Int.min % -1` ট্র্যাপ করে, কারণ ভাগফল `Int`-এ ধরে না।

**`isMultiple(of:)`।** `% == 0`-এর চেয়ে এটাই বলার মতো API। `0`, ঋণাত্মক সংখ্যা, এবং `Int.min` সামলায়।

```swift
let number = 120
number.isMultiple(of: 3)          // true
120.isMultiple(of: 3)             // লিটারালেও কল করা যায়
(-6).isMultiple(of: 3)            // true
0.isMultiple(of: 0)               // true
7.isMultiple(of: 0)               // false
Int.min.isMultiple(of: -1)        // true, ট্র্যাপ করে না
```

**সাইজ আর আলাদা টাইপ।** আজকের iPhone, iPad, Mac, Apple Watch-এ `Int` ৬৪-বিট। `Int` আর `Int64` একই সাইজ হলেও এক টাইপ নয়। এনোটেশন ছাড়া লিটারাল `Int` হয়।

| টাইপ | রেঞ্জ / সাইজ |
|---|---|
| `Int8` | −128…127 |
| `UInt8` | 0…255 |
| `Int32` | প্রায় ±2.1 বিলিয়ন |
| `Int64` | সব প্ল্যাটফর্মে ৬৪-বিট |
| `Int` | প্ল্যাটফর্মের শব্দদৈর্ঘ্য। Apple-এ এখন ৬৪-বিট |
| `Int.min` / `Int.max` | ওই প্ল্যাটফর্মের শেষ সীমা |

```swift
MemoryLayout<Int>.size     // 64-bit-এ 8
Int.bitWidth               // 64
```

কাউন্ট, ইনডেক্স, স্ক্রিন জুড়ে সাধারণ অঙ্কে `Int` ব্যবহার করুন। `UInt` ব্যবহার করুন বিট প্যাটার্ন বা এমন C API-তে, যেখানে আনসাইনড টাইপ বাধ্যতামূলক। `UInt` ঋণাত্মক মান আটকায় না: বিয়োগ করলে আন্ডারফ্লোতে ট্র্যাপ হয়। `Int` আর `UInt` মেশালে বারবার কনভার্ট করতে হয়।

বড় অ্যারেতে মেমরি কমাতে `Int8` বা `UInt8` রাখা যায়। অঙ্কের গতি সাধারণত বাড়ে না। ছোট টাইপে রেঞ্জ তাড়াতাড়ি শেষ হয়, তাই overflow বেশি ধরা পড়ে।

**কোনো ইমপ্লিসিট কনভার্সন নেই।**

```swift
let n = 10
let d = Double(n)
let back = Int(2.9)                 // 2, শূন্যের দিকে কাটে
let exact = Int(exactly: 2.0)       // Optional(2)
let notInt = Int(exactly: 2.5)      // nil

let small = Int8(exactly: 128)      // nil
let runtime = Int8(someInt)         // না ধরলে রানটাইমে ট্র্যাপ
```

লিটারাল না ধরলে কম্পাইল এরর: `let x: Int8 = 1000`। রানটাইম ভ্যালু না ধরলে `Int8(value)` ট্র্যাপ করে। নিরাপদ রূপ `Int8(exactly:)`।

**স্ট্রিং থেকে সংখ্যা।** ফেইলেবল ইনিশিয়ালাইজার। স্পেস কাটে না।

```swift
Int("42")              // Optional(42)
Int("-42")             // Optional(-42)
Int("42a")             // nil
Int(" 42")             // nil
Int("ff", radix: 16)   // Optional(255)
Int("1010", radix: 2)  // Optional(10)
```

লোকেলযুক্ত লেখায় (`1,50,000` বা `১২`) `NumberFormatter` ব্যবহার করুন।

**কপি সস্তা।** `Int` একটি শব্দ। কপি মানে সেই বিট কপি। `String`-এর মতো copy-on-write নেই, দরকারও নেই। `Hashable` ও `Comparable`, তাই ডিকশনারি কী, `Set` এলিমেন্ট, আর সর্টে সরাসরি ব্যবহার করা যায়।

**UI।** ইনডেক্স ও `count` হলো `Int`। ফ্রেম ও `CGFloat` আলাদা টাইপ। ৬৪-বিটে `CGFloat` আসলে `Double`।

```swift
let width = CGFloat(count) * 10
```

**JSON।** `Int` কোড হয় সংখ্যা হিসেবে। `2^53 - 1`-এর বড় মান JavaScript-এর `Number`-এ নিরাপদ নয়। সেই আইডি স্ট্রিং হিসেবে পাঠানোই নিরাপদ।

## MCQ

**1.** ৬৪-বিট iOS-এ কোনটা কম্পাইল হবে?

A. `let x: Int64 = 10 as Int`  
B. `let n = 10; let x: Int64 = n`  
C. `let x = Int64(10)`  
D. `let x: Int32 = 10; let y: Int = x`  

**Answer: C.** `Int`, `Int32`, `Int64` আলাদা টাইপ। সাইজ মিললেও ইমপ্লিসিট কনভার্সন নেই। `Int64(10)` লিটারাল থেকে স্পষ্টভাবে বানায়।

**2.** রিলিজ বিল্ডে `Int.max + 1` কী করে?

A. `Int.min`-এ র‍্যাপ করে  
B. `nil` দেয়  
C. overflow-এ ট্র্যাপ করে  
D. শুধু ডিবাগে ট্র্যাপ করে, রিলিজে র‍্যাপ করে  

**Answer: C.** সাধারণ `+` ডিবাগ ও রিলিজ দুই জায়গাতেই চেক করে। র‍্যাপ চাইলে `Int.max &+ 1`।

**3.** `-9 % 2` কত?

A. 1  
B. −1  
C. −4  
D. রানটাইম ট্র্যাপ  

**Answer: B.** `%` remainder। চিহ্ন ডিভিডেন্ডের। ভাগফল শূন্যের দিকে কাটে, তাই `-9 / 2` হয় `-4`, ভাগশেষ `-1`।

**4.** কোন কলটি ট্র্যাপ করবে?

A. `0.isMultiple(of: 0)`  
B. `Int.min.isMultiple(of: -1)`  
C. `Int.min % -1`  
D. `7.isMultiple(of: 0)`  

**Answer: C.** `%` ভাগফল বের করতে গিয়ে overflow করে। `isMultiple(of:)` এই কেসটি সামলায় এবং `true` দেয়। `7.isMultiple(of: 0)` হয় `false`।

**5.** `let same = 1_00__00___00____00` এর মান কত?

A. কম্পাইল এরর, কারণ `_` পরপর বসানো যায় না  
B. `100_000_000`  
C. `1`  
D. স্ট্রিং `"1_00__00___00____00"`  

**Answer: B.** আন্ডারস্কোর ফরম্যাটিং। কম্পাইলার সব `_` বাদ দিয়ে `100000000` রাখে।

**6.** `Int(exactly: 2.9)` আর `Int(2.9)` কী দেয়?

A. দুটোই `2`  
B. `nil` এবং `2`  
C. `3` এবং `2`  
D. দুটোই `nil`  

**Answer: B.** `Int.init` দশমিক অংশ ফেলে শূন্যের দিকে কাটে। `exactly` পূর্ণসংখ্যা না হলে `nil` দেয়।

**7.** দুইটা কনকারেন্ট টাস্ক `counter += 1` করলে কী ধরা নিরাপদ?

A. `Int` `Sendable`, তাই ইনক্রিমেন্ট নিরাপদ  
B. `+=` একটা CPU ইনস্ট্রাকশন, তাই অ্যাটমিক  
C. আপডেট হারাতে পারে  
D. কম্পাইলার অটো লক বসিয়ে দেয়  

**Answer: C.** ভ্যালু টাইপ কপি করা নিরাপদ। একই `var`-এ একসাথে লেখা ডেটা রেস। `+=` পড়া, যোগ, লেখা — এই তিন ধাপ।

## Short answer

**প্রশ্ন ১. Swift ইন্টিজার overflow-এ র‍্যাপ না করে ট্র্যাপ করে কেন?**  
ভুল র‍্যাপ পরে লজিক বাগ বা সিকিউরিটি বাগে পরিণত হয়। Swift-এর সাধারণ অপারেটর ফল চেক করে এবং ডিবাগ ও রিলিজ দুই জায়গাতেই ট্র্যাপ করে, তাই ব্যর্থতা খারাপ অঙ্কের পাশেই ধরা পড়ে। হ্যাশ বা বিট মাস্কে র‍্যাপই স্পেক হলে `&+`, `&-`, `&*` ব্যবহার করুন। কলারকে সিদ্ধান্ত নিতে হলে `addingReportingOverflow` ব্যবহার করুন এবং overflow ফ্ল্যাগ পড়ুন।

**প্রশ্ন ২. `%` আর modulo কীভাবে আলাদা?**  
`%` শূন্যের দিকে ভাগের remainder। চিহ্ন বাঁ পাশের অপারেন্ডের থাকে। `-9 % 2` হয় `-1`। ধনাত্মক ভাজকের mathematical modulo কখনো ঋণাত্মক হয় না। আরেকটি বিষয়: `Int.min % -1` ট্র্যাপ করে, কারণ ভাগফল `Int`-এ ধরে না। কোনো সংখ্যা গুণিতক কি না, সেটা `isMultiple(of:)` দিয়ে দেখুন। শূন্য ছাড়া অন্য সংখ্যাকে `0` দিয়ে ভাগের গুণিতক ধরলে `false` আসে। `0.isMultiple(of: 0)` হয় `true`। `Int.min`-এ এটা ট্র্যাপ করে না।

**প্রশ্ন ৩. মান ঋণাত্মক হতে না পারলেও API গাইডলাইন `Int` বেছে নেয় কেন?**  
`UInt`-ও আন্ডারফ্লোতে ট্র্যাপ করে। “অবশ্যই ধনাত্মক” এই নিয়ম টাইপ সিস্টেমে কার্যকরভাবে প্রকাশ পায় না। `Int` আর `UInt`-এর মধ্যে ইমপ্লিসিট কনভার্সন নেই, তাই মেশালে কোডজুড়ে কাস্ট ছড়িয়ে পড়ে। কাউন্ট, ইনডেক্স, `Array.count` — সব `Int`। C স্ট্রাকট, ফাইল ফরম্যাট, বা সত্যিকারের বিট প্রস্থ (যেমন বাইটের জন্য `UInt8`) হলে ফিক্সড-উইডথ বা আনসাইনড টাইপ ব্যবহার করুন।

**প্রশ্ন ৪. অপ্রত্যাশিত ট্রাঙ্কেশন ছাড়া সংখ্যা কনভার্ট করবেন কীভাবে?**  
ম্যান্টিসার ৫৩ বিটের ভিতরের পূর্ণসংখ্যায় `Double(int)` নির্ভুল। `Int(double)` শূন্যের দিকে কাটে, তাই `Int(-2.9)` হয় `-2`। মান পূর্ণসংখ্যা না হলে, বা না ধরলে, `Int(exactly:)` `nil` দেয়। প্রস্থের ক্ষেত্রেও একই ভাগ: রানটাইম মান রেঞ্জের বাইরে হলে `Int8(someInt)` ট্র্যাপ করে। `Int8(exactly:)` অপশনাল দেয়। যে লিটারাল ধরে না, যেমন `let x: Int8 = 1000`, সেটা কম্পাইল এরর।

**প্রশ্ন ৫. iPhone-এ `Int` আর `Int64` একই সাইজ। বদলে বদলে ব্যবহার করা যায় না কেন?**  
একই সাইজ প্ল্যাটফর্মের ঘটনা। টাইপ তবু আলাদা, এবং Swift-এ সংখ্যার ইমপ্লিসিট কনভার্সন নেই। ৬৪-বিট ডিভাইসে `MemoryLayout<Int>.size` ও `MemoryLayout<Int64>.size` দুটোই ৮, তবু `let x: Int64 = someInt` কম্পাইল হয় না। `Int` পয়েন্টার-সাইজড, তাই প্রস্থ প্ল্যাটফর্ম অনুসারে। `Int64` সব প্ল্যাটফর্মে ৬৪-বিট থাকে। সীমানায় `Int64(someInt)` বা `Int(exactly: someInt64)` দিয়ে কনভার্ট করুন।

**প্রশ্ন ৬. ইউজার যা টাইপ করেছে, সেই সংখ্যা কীভাবে পার্স করবেন?**  
`Int(text)` অপশনাল চিহ্ন ও র‍্যাডিক্স নেয়। `"42a"` বা আশেপাশের স্পেসে `nil` দেয়। গ্রুপিং সেপারেটর বা বাংলা অঙ্ক বোঝে না। সেগুলোর জন্য ইউজারের লোকেলসহ `NumberFormatter` ব্যবহার করুন। ভ্যালিডেশন সফল না হওয়া পর্যন্ত ফল অপশনাল রাখুন। “নেই” বোঝাতে `0` ব্যবহার করবেন না।

দশমিক সংখ্যা (`Double`) — ইন্টারভিউ নোট (Google Doc-এ কপি করার জন্য)

এই টপিক আবার সাজানো হলো। ৩০ সেকেন্ডের উত্তর এবং Short Answer এখন বাংলায়। পরের টপিকেও এই দুই অংশ বাংলায় থাকবে।

# 4. Decimal Numbers (`Double`, `Float`, `Decimal`)

## ৩০ সেকেন্ডে বলার উত্তর

ডট থাকলে লিটারাল `Double`। এটা IEEE 754 বাইনারি ফ্লোটিং পয়েন্ট। `0.1 + 0.2` আর `0.3` এক জিনিস নয়, কারণ বেশিরভাগ দশমিক ভগ্নাংশ বাইনারিতে নেই। `Int` আর `Double` আলাদা টাইপ। একবার টাইপ হয়ে যাওয়া মান নিজে থেকে বদলায় না। লিটারাল পলিমরফিক, তাই `1 + 2.0` কম্পাইল হয়, আর টাইপ করা `a + b` হয় না। পূর্ণসংখ্যায় শূন্য দিয়ে ভাগ করলে ট্র্যাপ হয়। ভাসমান সংখ্যায় শূন্য দিয়ে ভাগ করলে `inf` বা `NaN` আসে, আর `NaN` নিজের সমান নয়। আজকের ৬৪-বিট iPhone-এ `CGFloat` হলো `Double`-এর typealias। টাকা রাখতে `Decimal` বা পূর্ণসংখ্যার সেন্ট ব্যবহার করুন।

## যে পয়েন্টগুলো বলতে হবে

**লিটারাল দেখে টাইপ ঠিক হয়।** ডট থাকলে `Double`, না থাকলে `Int`। ডটের পরে শুধু `0` থাকলেও `Double`।

```swift
let double1 = 3.1
let double2 = 3131.3131
let double3 = 3.0      // Double
let int1 = 3           // Int
let tiny = 1.5e-10
let grouped = 1_000.25
```

`let x = 0.1` সবসময় `Double`। `Float` চাইলে কনটেক্সট দিন:

```swift
let f: Float = 0.1
let g = Float(0.1)
```

**বাইনারি ভগ্নাংশ।** কম্পিউটার সাইন, এক্সপোনেন্ট, ম্যান্টিসা রাখে। দশমিক পয়েন্ট সংখ্যার মাত্রা অনুযায়ী সরে, তাই নাম floating point। `1/3` যেমন দশমিকে শেষ হয় না, `0.1` তেমন বাইনারিতে শেষ হয় না। কাছাকাছি মান জমা হয়।

```swift
let number = 0.1 + 0.2
print(number)   // 0.30000000000000004
```

UI পয়েন্টে এই ত্রুটি প্রায় ধরা যায় না। সমতা চেক, টাকা, আর অনেকগুলো ছোট সংখ্যা যোগে এটাই বাগ।

**টাইপ একবার ঠিক হলে আর বদলায় না।**

```swift
var name = "Nicolas Cage"
name = "John Travolta"   // বৈধ
name = 57                // কম্পাইল এরর
```

**টাইপ সেফটি: টাইপ করা `Int` আর `Double` এক অঙ্কে মেশে না।**

```swift
let a = 1
let b = 2.0
let c = a + b            // এরর

let asInt = a + Int(b)   // Int। দশমিক কাটে, শূন্যের দিকে
let asDouble = Double(a) + b
```

`Int(2.9)` হয় `2`। `Int(-2.9)` হয় `-2`। `NaN`, `inf`, বা `Int` রেঞ্জের বাইরের `Double` দিয়ে `Int(...)` করলে রানটাইমে ট্র্যাপ। নিরাপদ রূপ `Int(exactly:)` — পূর্ণসংখ্যা না হলে বা না ধরলে `nil`।

**লিটারাল পলিমরফিক। টাইপ করা ভ্যালু নয়।** এখানেই ইন্টারভিউয়ের ফাঁদ।

```swift
let ok = 1 + 2.0         // Double। 1 এখনো টাইপ পায়নি
var rating = 5.0
rating *= 2              // 2 লিটারাল, Double হিসেবে ম্যাচ করে

let n: Int = 1
// n + 2.0               // এরর। n ইতিমধ্যে Int
```

`Float` আর `Double`-ও আলাদা। `Float(d)` বা `Double(f)` লাগে।

**একই অপারেটর।** `+ - * /` এবং `+= -= *= /=`। বাঁ পাশ `var`।

```swift
var rating = 5.0
rating *= 2              // 10.0
```

**`Int` আর `Double`-এ শূন্য দিয়ে ভাগ আলাদা জিনিস।**

```swift
let inf = 1.0 / 0.0          // inf, ট্র্যাপ নয়
let negInf = -1.0 / 0.0      // -inf
let nan = 0.0 / 0.0          // nan
inf.isInfinite               // true
nan.isNaN                    // true

nan == nan                   // false
nan < 1                      // false
nan > 1                      // false
0.0 == -0.0                  // true
```

`NaN` যেকোনো অঙ্কে ছড়িয়ে পড়ে। `==` দিয়ে `NaN` চেক করা যায় না। `isNaN` ব্যবহার করুন। `Set` বা `Dictionary` কী হিসেবে `Double` রাখলে `NaN` ও `-0.0` সমস্যার জায়গা, কারণ `NaN == NaN` মিথ্যা। JSON-এ `NaN` ও `inf` বৈধ সংখ্যা নয়। `JSONEncoder` এগুলো এনকোড করতে ব্যর্থ হয়।

`Int`-এর `+` overflow-এ ট্র্যাপ করে। `Double` overflow-এ `inf` হয়।

```swift
Double.greatestFiniteMagnitude * 2   // inf
```

**প্রস্থ।**

| টাইপ | সাইজ | দশমিক অঙ্ক (মোটামুটি) |
|---|---|---|
| `Float` | 32-bit | প্রায় 7 |
| `Double` | 64-bit | প্রায় 15–16 |
| `Float16` | 16-bit | ML / গ্র্যাফিক্স |
| `CGFloat` | প্ল্যাটফর্ম | 64-bit-এ `Double`-এর সমান |

`Double`-এর ম্যান্টিসা ৫৩ বিট। `2^53`-এর ওপরের সব পূর্ণসংখ্যা `Double`-এ নেই। বড় আইডি `Double` দিয়ে রাউন্ড-ট্রিপ করলে বদলে যায়।

খুব বড় সংখ্যায় ছোট যোগ হারিয়ে যায়:

```swift
let big = 1e16
big + 1.0 == big     // true
```

**তুলনা।** `==` দিয়ে দশমিক ভাসমান সংখ্যা মেলানো যায় না। ছোট মানে অ্যাবসলিউট টলারেন্স। বড় মানে রিলেটিভ টলারেন্স বা ULP।

```swift
func nearlyEqual(_ a: Double, _ b: Double, tolerance: Double = 1e-9) -> Bool {
    if a.isNaN || b.isNaN { return false }
    return abs(a - b) <= tolerance * max(1, max(abs(a), abs(b)))
}
```

`stride(from: 0, through: 1, by: 0.1)` ঠিক দশ ভাগে ভাগে নাও। ধাপের সংখ্যা জানা থাকলে ইন্টিজার লুপ চালিয়ে ভাগ করুন।

**রাউন্ডিং।** ডিফল্ট IEEE নিয়ম `.toNearestOrEven` — banker's rounding।

```swift
(2.5).rounded()                              // 2
(3.5).rounded()                              // 4
(2.5).rounded(.toNearestOrAwayFromZero)      // 3
(2.9).rounded(.towardZero)                   // 2
```

টাকার হিসাবে এই ডিফল্ট চুপচাপ এক সেন্ট সরাতে পারে। নিয়ম স্পষ্ট করে দিন।

**টাকা `Double`-এ নয়।**

```swift
let alreadyWrong = Decimal(0.1)                 // Double-এর ত্রুটি আগেই ঢুকেছে
let price = Decimal(string: "19.99")!          // ঠিক শুরু
```

`Decimal` দশমিক ভাসমান সংখ্যা। প্রায় ৩৮ দশমিক অঙ্ক। Foundation-এর স্ট্রাকট। অঙ্ক সফটওয়্যারে হয়, তাই গ্র্যাফিক্সের জন্য ধীর। ইনিশিয়ালাইজার স্ট্রিং বা পূর্ণসংখ্যা থেকে দিন।

দৈনন্দিন মুদ্রায় আরও সরল মডেল: সেন্ট `Int`-এ রাখুন। `1999` মানে ১৯.৯৯। ভাগের সময় রাউন্ডিং নিয়ম আলাদা করে লিখুন।

**`CGFloat`।** Swift 5.5 থেকে আজকের 64-bit iPhone ও Mac-এ `CGFloat` হলো `Double`-এর typealias। একই টাইপ, একই প্রিসিশন। পাঠ্যের “যেখানে `CGFloat` লাগে সেখানে `Double` দিন” এই কারণেই চলে। `arm64_32` ও পুরোনো 32-bit-এ `CGFloat` 32-bit, `Float`-এর প্রস্থ। সেই ABI-তে typealias নয়। পোর্টেবল API-তে রূপান্তর স্পষ্ট রাখুন। লেআউটে `CGFloat` দেখলে আলাদা গণিতের টাইপ ভেবে দ্বিগুণ কনভার্ট করার দরকার নেই। `Float`-এ নামিয়ে নিলে প্রিসিশন কমে।

## MCQ

**1.** `print(0.1 + 0.2)` কী ছাপে?

A. `0.3`  
B. `0.30000000000000004`  
C. কম্পাইল এরর  
D. রানটাইম ট্র্যাপ  

**Answer: B.** `0.1` ও `0.2` বাইনারিতে অসীম ভগ্নাংশ। যোগফল `0.3`-এর কাছাকাছি একটি রিপ্রেজেন্টেবল মান।

**2.** কোন লাইন কম্পাইল হবে?

A. `let a = 1; let b = 2.0; let c = a + b`  
B. `let c = 1 + 2.0`  
C. `let f: Float = 1; let d: Double = 2; let c = f + d`  
D. `var name = "Nicolas Cage"; name = 57`  

**Answer: B.** লিটারাল এখনো `Int` হয়নি, তাই `Double` অঙ্কে বসতে পারে। `a` ও `f` আগেই টাইপ পেয়ে গেছে।

**3.** `1.0 / 0.0` কী দেয়?

A. `0`  
B. রানটাইম ট্র্যাপ, `Int`-এর মতো  
C. `inf`  
D. `nil`  

**Answer: C.** ভাসমান ভাগ ট্র্যাপ করে না। `1.0 / 0.0` হয় `inf`, `0.0 / 0.0` হয় `NaN`।

**4.** `Double.nan == Double.nan` কী?

A. `true`  
B. `false`  
C. কম্পাইল এরর  
D. `inf`  

**Answer: B.** `NaN` নিজের সমান নয়। চেক `isNaN` দিয়ে। `>`, `<`, `==` সব মিথ্যা।

**5.** `(2.5).rounded()` কত?

A. `3`  
B. `2`  
C. `2.5`  
D. ট্র্যাপ  

**Answer: B.** ডিফল্ট `.toNearestOrEven`। `2` জোড়, তাই `2.5` হয় `2`। `3.5` হয় `4`।

**6.** টাকার `19.99` কোথায় রাখবেন?

A. `Double(19.99)`  
B. `Decimal(19.99)`  
C. `Decimal(string: "19.99")` অথবা সেন্ট হিসেবে `Int(1999)`  
D. `Float`, কারণ ছোট মানে ত্রুটি কম  

**Answer: C.** `Decimal(19.99)` আগে `Double` বানায়, ত্রুটি ঢুকে যায়। স্ট্রিং বা পূর্ণসংখ্যার সেন্ট নিরাপদ।

**7.** আজকের iPhone-এ `CGFloat` কী?

A. সবসময় `Float`  
B. `Double`-এর typealias  
C. `Decimal`  
D. 128-bit মান  

**Answer: B.** 64-bit iOS/macOS-এ Swift 5.5 থেকে typealias। `arm64_32`-তে `CGFloat` 32-bit থাকে।

**8.** `let big = 1e16` হলে `big + 1.0 == big` কেন `true`?

A. কম্পাইলার যোগ মুছে দেয়  
B. এই মাত্রায় ULP ১-এর চেয়ে বড়, তাই `1.0` ম্যান্টিসায় জায়গা পায় না  
C. `Double` ১৬ অঙ্কের পর কাটে সবসময়  
D. `==` ভাসমান সংখ্যায় সবসময় `true`  

**Answer: B.** ৫৩-বিট ম্যান্টিসা। সংখ্যা যত বড়, দুই প্রতিবেশীর ফাঁক তত বড়।

## Short answer

**প্রশ্ন ১. `0.1 + 0.2` আর `0.3` সমান নয় কেন?**  
`Double` হলো IEEE 754 বাইনারি ফ্লোটিং পয়েন্ট: এক বিট সাইন, এগারো বিট এক্সপোনেন্ট, বাহান্ন বিট এক্সপ্লিসিট ম্যান্টিসা। `0.1`-এর মতো মান বেস ২-এ পুনরাবৃত্ত ভগ্নাংশ, তাই জমা হয় সবচেয়ে কাছের যে সংখ্যাটি রাখা যায়। দুটি আনুমানিক মান যোগ করলে আরেকটি আনুমানিক মান আসে। Swift ছাপে `0.30000000000000004`। গ্র্যাফিক্সে এই ফরম্যাট দ্রুত এবং যথেষ্ট। হুবহু দশমিক অঙ্কের জন্য এটা ভুল হাতিয়ার।

**প্রশ্ন ২. `let a = 1; let b = 2.0; a + b` ব্যর্থ হয়, অথচ `1 + 2.0` চলে কেন?**  
Swift `a`-কে `Int` এবং `b`-কে `Double` ধরে। টাইপ হয়ে যাওয়া সংখ্যা নিজে থেকে অন্য টাইপে যায় না, তাই এই যোগের কোনো অপারেটর মেলে না। খালি লিটারাল তখনো পলিমরফিক। `1 + 2.0`-এ কম্পাইলার `1`-কে `Double` বানিয়ে এক্সপ্রেশন কম্পাইল করে। একই নিয়মে `rating` যখন `Double`, `rating *= 2` বৈধ। রূপান্তর স্পষ্ট হতে হবে: `Double(a) + b`। শূন্যের দিকে কাটা গ্রহণযোগ্য হলে `a + Int(b)`।

**প্রশ্ন ৩. দুটি `Double` কীভাবে তুলনা করবেন?**  
অঙ্ক থেকে আসা ফলে `==` ব্যবহার করবেন না। আগে `NaN` বাদ দিন। তারপর রিলেটিভ টলারেন্স নিন: বড় ম্যাগনিটিউড দিয়ে স্কেল করা পার্থক্য, আর শূন্যের কাছেও যাতে কাজ করে এমন একটি ছোট তল। লম্বা ধাপের জন্য `stride(by: 0.1)` এড়িয়ে ইন্টিজার লুপ চালিয়ে একবার ভাগ করুন, কারণ ত্রুটি জমতে থাকে। শুধু অ্যাবসলিউট এপসিলন বড় সংখ্যায় বিভ্রান্ত করে, কারণ দুটি রাখা-যায়-এমন মানের ফাঁক সংখ্যার সাথে বাড়ে।

**প্রশ্ন ৪. টাকা কীভাবে রাখবেন?**  
ছোট একক `Int`-এ রাখুন, যেমন সেন্ট। অথবা স্ট্রিং বা পূর্ণসংখ্যা থেকে বানানো `Decimal` রাখুন। `Decimal(19.99)` আগে থেকেই ভুল, কারণ এই ইনিশিয়ালাইজার `Double` নেয় এবং বাইনারি ত্রুটি ভিতরে ঢুকে যায়। `Decimal` প্রায় আটত্রিশ দশমিক অঙ্ক রাখে এবং স্পষ্ট রাউন্ডিং মোড নেয়। এটা `Double`-এর চেয়ে অনেক ধীর, তাই দাম ও টোটালে থাকবে, ফ্রেমপ্রতি অঙ্কে নয়। ডিফল্ট `rounded()` banker's rounding ব্যবহার করে, তাই `2.5` হয় `2`। টাকায় রাউন্ডিং নিয়ম স্পষ্ট করে লিখুন।

**প্রশ্ন ৫. বর্তমান iPhone-এ `CGFloat` কী, আর `Double`-এর সাথে পার্থক্য কোথায়?**  
৬৪-বিট iOS ও macOS-এ `CGFloat` হলো `Double`-এর typealias। লেআউট কোড আর Swift-এর সংখ্যা একই টাইপ, একই প্রিসিশন। তাই যেখানে `CGFloat` চায়, সেখানে `Double` দেওয়া যায়। `arm64_32` ও পুরোনো 32-bit ABI-তে `CGFloat` 32-bit এবং `Float`-এর সমান। গ্র্যাফিক্সের মান চুপচাপ `Float`-এ নামাবেন না। ৬৪-বিট ইন্টিজার আইডিও `Double`-এ রাখবেন না: `2^53`-এর উপরে সব পূর্ণসংখ্যা রাখা যায় না।

**প্রশ্ন ৬. overflow আর শূন্য দিয়ে ভাগ `Int`-এর চেয়ে কীভাবে আলাদা?**  
চেক করা পূর্ণসংখ্যার অঙ্ক ট্র্যাপ করে। `Double` overflow-এ `inf` হয়। শূন্য দিয়ে ভাগ ট্র্যাপ করে না: `1.0 / 0.0` হয় `inf`, `0.0 / 0.0` হয় `NaN`। `NaN` সবকিছুর সাথে অসমান, নিজের সাথেও। চেক `isNaN`। `+0` আর `-0` সমান হিসেবে তুলনা হয়। `JSONEncoder` দিয়ে `NaN` বা infinity এনকোড করা যায় না, কারণ JSON সংখ্যায় এই মান দুটি নেই। `NaN`, infinity, বা রেঞ্জের বাইরের `Double`কে `Int(...)` দিয়ে বদলালে ট্র্যাপ হয়। `Int(exactly:)` দিলে `nil` আসে।

# 5. Truth (`Bool`)

## ৩০ সেকেন্ডে বলার উত্তর

`Bool` শুধু `true` বা `false` রাখে। `hasSuffix` বা `isMultiple(of:)` যে চেক করে, তার ফল এই টাইপেই আসে। অঙ্কের অপারেটর নেই। `!` মান উল্টে নতুন `Bool` দেয়। `toggle()` একই কাজ জায়গাতেই করে, তাই বাঁ পাশ `var` হতে হয়। Swift-এ truthiness নেই: `if` শুধু `Bool` নেয়, `Int` বা `String` নয়। `&&` আর `||` শর্ট-সার্কিট করে, কারণ ডান পাশ `@autoclosure`। `Bool?`-এ তিন অবস্থা: `true`, `false`, `nil`। `if let` করলে `false`-ও ভিতরে ঢোকে। `nil` মানে `false` নয়।

## যে পয়েন্টগুলো বলতে হবে

**সরাসরি বানানো যায়, এক্সপ্রেশন থেকেও।** নাম George Boole-এর থেকে। লজিকের দুটি মান।

```swift
let goodDogs = true
let gameOver = false

let filename = "paris.jpg"
let isJPEG = filename.hasSuffix(".jpg")   // Bool

let number = 120
let isMultiple = number.isMultiple(of: 3)  // true
```

`switch` করলে `true` আর `false` মিলিয়েই exhaustive। `default` লাগে না।

**`!` নতুন মান দেয়। `toggle()` নিজেকে বদলায়।**

```swift
var isAuthenticated = false
isAuthenticated = !isAuthenticated
print(isAuthenticated)          // true

isAuthenticated = !isAuthenticated
print(isAuthenticated)          // false

var gameOver = false
gameOver.toggle()
print(gameOver)                 // true
```

`toggle()` আসলে `self = !self`। লম্বা নামে পড়া সহজ। `let`-এর ওপর কল হয় না, কারণ মেথড `mutating`। `let`-এর উল্টো মান নতুন কনস্ট্যান্টে রাখুন: `let signedOut = !isAuthenticated`।

`!` দুই রকম। প্রিফিক্স `!` মানে Bool NOT। পোস্টফিক্স `!` মানে Optional ফোর্স-আনর‍্যাপ।

```swift
let flag: Bool? = true
let flipped = !flag!     // আগে আনর‍্যাপ, তারপর NOT। ফল false
```

`flag` যদি `nil` হয়, `flag!` ট্র্যাপ করে। `Bool?`-এ সরাসরি `!flag` কম্পাইল হয় না।

**অঙ্ক নেই। ০/১-ও বসানো যায় না।** `true + true` কম্পাইল হয় না। `Int(true)`-ও হয় না। দরকার হলে টার্নারি: `isOn ? 1 : 0`।

**Truthiness নেই।** C, JavaScript, Python-এ সংখ্যা বা স্ট্রিং শর্ত হতে পারে। Swift-এ `if` ও `guard` শুধু `Bool` চায়।

```swift
let count = 0
// if count { }          // এরর
if count != 0 { }

let name = "Ted"
// if name { }           // এরর
if !name.isEmpty { }
```

**শর্ট-সার্কিট।** `&&` ও `||` স্ট্যান্ডার্ড লাইব্রেরিতে `@autoclosure`। বাঁ পাশ দেখে ডান পাশ একেবারেই চলে না।

```swift
func expensive() -> Bool {
    print("called")
    return true
}

false && expensive()    // expensive() কল হয় না
true || expensive()     // expensive() কল হয় না
```

বাউন্ড চেক এই কারণেই বাঁ পাশে থাকে:

```swift
if index >= 0 && index < items.count && items[index].isSelected {
    // index বৈধ না হলে সাবস্ক্রিপ্ট চলে না
}
```

প্রিসিডেন্স: `!`, তারপর `&&`, তারপর `||`। `a || b && c` মানে `a || (b && c)`। মেশানো এক্সপ্রেশনে বন্ধনী দিন।

দুই পাশই চালাতে হলে আগে আলাদা `let`-এ নিন। সাইড ইফেক্ট `&&`-এর ডান পাশে লুকোবেন না। `assert`-এর শর্তও অটোক্লোজার। রিলিজে `assert` উঠে যায়, তাই দরকারি কাজ সেখানে রাখবেন না। `precondition` ডিফল্ট রিলিজে থাকে। `-Ounchecked`-এ সেটাও ওঠে।

**`Bool?` তিন অবস্থা।** এটাই সবচেয়ে ঘনঘন বাগ।

| মান | `== true` | `== false` | `if let x = value` |
|---|---|---|---|
| `nil` | ঢোকে না | ঢোকে না | ঢোকে না |
| `false` | ঢোকে না | ঢোকে | ঢোকে |
| `true` | ঢোকে | ঢোকে না | ঢোকে |

```swift
let isAdmin: Bool? = false

if let isAdmin {
    // false হলেও এখানে ঢোকে। nil হলেই শুধু বাদ
}

if isAdmin == true {
    // শুধু সত্যিকারের true
}
```

অপশনাল চেইনে `nil` আর `false` এক পথে পড়ে:

```swift
if user?.isAdmin == true {
    // user না থাকলে এক্সপ্রেশন nil। nil == true মিথ্যা
}
```

“জানি না” আর “না” আলাদা রাখতে `Bool?` রাখুন। শুধু হ্যাঁ/না হলে `Bool` রাখুন, `nil`-কে `false` বানিয়ে ফেলবেন না।

**জমা ও ব্রিজ।** `Bool` একটি ভ্যালু টাইপ। `MemoryLayout<Bool>.size` হয় ১। `[Bool]`-এ প্রতিটি এলিমেন্ট এক বাইট, এক বিট নয়। অনেক ফ্ল্যাগ একসাথে একটি C বিটমাস্ক হলে `OptionSet`। আলাদা আলাদা অবস্থা থাকলে আলাদা `Bool`।

`Codable` JSON-এ `true`/`false` লেখে। `1` বা `0` ডিফল্টে `Bool` হয় না, `typeMismatch` হয়। Foundation-এ `Bool` Objective-C `BOOL`-এ ব্রিজ হয়। `Bool` নিজে `Sendable`। একই `var` দুই থ্রেডে লিখলে ডেটা রেস থাকে।

SwiftUI-তে `@State var isPresented = false`, তারপর `isPresented.toggle()`। `disabled(!isFormValid)` শুধু তখনই নিরাপদ, যখন মান সত্যিই `Bool`, `Bool?` নয়।

## MCQ

**1.** কোনটা কম্পাইল হবে?

A. `let n = 3; if n { print(n) }`  
B. `let s = "a"; if s { print(s) }`  
C. `let ok = !filename.hasSuffix(".png")`  
D. `let sum = true + true`  

**Answer: C.** `if` শুধু `Bool` নেয়। `Bool`-এ `+` নেই। `hasSuffix` `Bool` দেয়, `!` সেটা উল্টে দেয়।

**2.** `false && expensive()` হলে `expensive()` কী হয়? ফাংশন `print("called")` করে।

A. `"called"` ছাপে  
B. কলই হয় না  
C. রিলিজ বিল্ডে কল হয়, ডিবাগে হয় না  
D. কম্পাইল এরর, কারণ `&&` দুই পাশ চায়  

**Answer: B.** ডান পাশ `@autoclosure`। বাঁ পাশ `false` হলে ডান পাশ ইভ্যালুয়েট হয় না।

**3.** `let isAdmin: Bool? = false`। কোন ব্লকটি চলবে?

A. `if let isAdmin { grant() }` — `grant()` কল হয়  
B. `if isAdmin == true { grant() }` — `grant()` কল হয়  
C. `if isAdmin == false { }` — এই ব্লক চলে না  
D. `if !isAdmin { }` — কম্পাইল হয়ে `grant` এড়ায়  

**Answer: A.** `if let` শুধু `nil` বাদ দেয়। `false` একটি আসল মান, তাই ব্লকে ঢোকে। `!isAdmin` কম্পাইল হয় না। `== true` এই কেসে ঢোকে না।

**4.** `var gameOver = false` এর পর কোনটা বৈধ?

A. `let fixed = false; fixed.toggle()`  
B. `gameOver.toggle()`  
C. `gameOver = gameOver + true`  
D. `let n = Int(gameOver)`  

**Answer: B.** `toggle()` মিউটেটিং। `let`-এ চলে না। `Int` আর `Bool` নিজে থেকে বদলায় না।

**5.** `let flag: Bool? = true`। `!flag!` কী?

A. কম্পাইল এরর  
B. `true`  
C. `false`  
D. `nil`  

**Answer: C.** পোস্টফিক্স `!` আগে আনর‍্যাপ করে। প্রিফিক্স `!` তারপর NOT করে। `flag` যদি `nil` হতো, আনর‍্যাপ ট্র্যাপ করত।

**6.** `index` সীমার বাইরে। কোন ক্রমটি সাবস্ক্রিপ্টের আগে থামে?

A. `items[index].isSelected && index < items.count`  
B. `index >= 0 && index < items.count && items[index].isSelected`  
C. `items[index].isSelected || index < items.count`  
D. দুটোই একই, কারণ `&&` সবসময় দুই পাশ চালায়  

**Answer: B.** `&&` বাঁ থেকে ডানে থামে। ইনডেক্স বৈধ হওয়ার পরই সাবস্ক্রিপ্ট চলে। উল্টো ক্রমে আগেই ক্র্যাশ।

**7.** সার্ভার JSON-এ `{"admin": 1}`। `admin`-কে `Bool` ডিকোড করলে কী হয়?

A. `true` হয়  
B. `false` হয়  
C. `typeMismatch`  
D. `1` একটি বৈধ `Bool` লিটারাল  

**Answer: C.** `Codable` শুধু JSON `true`/`false` নেয়। `0`/`1` ডিফল্টে `Bool` হয় না।

## Short answer

**প্রশ্ন ১. `!` আর `toggle()` কীভাবে আলাদা?**  
`!` একটি নতুন `Bool` দেয়। পুরোনো বাইন্ডিং বদলায় না, তাই `let`-এর উল্টো মান আলাদা কনস্ট্যান্টে রাখা যায়। `toggle()` মিউটেটিং মেথড। ভিতরে `self = !self` চলে, তাই টার্গেট `var` হতে হয়। লম্বা প্রপার্টির নামে `isPresented.toggle()` পড়া সহজ। ফল একই। পোস্টফিক্স `!` অন্য জিনিস: সেটা Optional আনর‍্যাপ করে। `Bool?`-এ প্রিফিক্স `!` সরাসরি বসানো যায় না।

**প্রশ্ন ২. Swift-এ `if count` চলে না কেন?**  
`if`, `guard`, `while` শুধু `Bool` চায়। `Int`, `String`, অপশনাল — এগুলোর আলাদা “খালি” মান আছে, কিন্তু সেগুলো শর্ত হিসেবে বসে না। `0` বা `""` দিয়ে চুপচাপ মিথ্যা বানিয়ে ফেললে ভুল লুকিয়ে যায়। শর্ত স্পষ্ট লিখতে হয়: `count != 0`, `!name.isEmpty`। একই নিয়মে `true + true` বা `Int(true)` কম্পাইল হয় না। বিট দরকার হলে `isOn ? 1 : 0`।

**প্রশ্ন ৩. `&&` কীভাবে ক্র্যাশ আটকায়?**  
ডান পাশ `@autoclosure`। বাঁ পাশ `false` হলে ডান পাশ কল হয় না। `||`-এ বাঁ পাশ `true` হলে ডান পাশ চলে না। তাই `index >= 0 && index < items.count && items[index].isSelected` নিরাপদ। সাবস্ক্রিপ্ট আগে লিখলে শর্ট-সার্কিট আর বাঁচাতে পারে না। প্রিসিডেন্সে `&&` আগে, `||` পরে। দুই পাশের সাইড ইফেক্ট দরকার হলে আগে আলাদা ভেরিয়েবলে হিসাব করুন। `assert`-এর ভিতরে দরকারি কাজ রাখবেন না। রিলিজে `assert` সরে যায়।

**প্রশ্ন ৪. `Bool?`-এ `if let` বিপজ্জনক কোথায়?**  
`if let` শুধু `nil` সরায়। `false` একটি বৈধ মান, তাই ব্লকের ভিতরে ঢোকে। `isAdmin` মিথ্যা হলেও অ্যাক্সেস দিয়ে ফেলা এই বাগ। শুধু হ্যাঁ ধরতে `if isAdmin == true` লিখুন। অথবা `if let isAdmin, isAdmin`। `nil == false` মিথ্যা, তাই “মান নেই” আর “না” এক জিনিস নয়। `user?.isAdmin == true` তখনই ঢোকে, যখন ইউজার আছে এবং অ্যাডমিন। ইউজার না থাকলে এক্সপ্রেশন `nil`।

**প্রশ্ন ৫. অনেকগুলো হ্যাঁ/না কীভাবে রাখবেন?**  
স্বাধীন অবস্থা আলাদা `Bool` থাকে। যেমন `isLoggedIn` আর `hasSeenOnboarding`। একসাথে পাঠানো অপশনের সেট, বা C-এর বিটমাস্ক, হলে `OptionSet`। `[Bool]` প্রতি মানে এক বাইট রাখে, এক বিট নয়। JSON-এ `Bool` হয় `true`/`false`। সার্ভার `0`/`1` পাঠালে ডিফল্ট `Codable` `typeMismatch` দেয়। তখন ডিকোডিং আলাদা করে লিখুন। Objective-C সীমানায় `Bool` ব্রিজ হয় `BOOL`-এ।

স্ট্রিং জোড়া — ইন্টারভিউ নোট (Google Doc-এ কপি করার জন্য)

# 6. Joining Strings (`+`, interpolation)

## ৩০ সেকেন্ডে বলার উত্তর

`+` দুটি `String` জুড়ে নতুন `String` দেয়। `Int` বা `Double`-এর সাথে সরাসরি জোড়া যায় না। চেইন বাঁ-অ্যাসোসিয়েটিভ, তাই প্রতি ধাপের ফল আলাদা স্ট্রিং। লুপে `s = s + part` বারবার বাফার কপি করে। `+=`, `append`, `joined(separator:)`, বা একটি ইন্টারপোলেশন এক বাফারে জমা করে। `\(value)` ভিতরে `String(describing:)`। অপশনাল চুপচাপ `Optional("...")` বানিয়ে দেয়। সব অংশ কম্পাইল-টাইম লিটারাল হলে অপটিমাইজার `+` ভাঁজ করতে পারে। ইউজারকে দেখানো সংখ্যা আর মেশিন-কী আলাদাভাবে ফরম্যাট করতে হয়।

## যে পয়েন্টগুলো বলতে হবে

**`+` শুধু স্ট্রিংয়ের সাথে স্ট্রিং।** একে operator overloading বলে: একই `+` টাইপ দেখে আলাদা কাজ করে। `String`-এ যোগ, `Int`/`Double`-এ অঙ্ক।

```swift
let firstPart = "Hello, "
let secondPart = "world!"
let greeting = firstPart + secondPart

let people = "Haters"
let action = "hate"
let lyric = people + " gonna " + action
print(lyric)   // Haters gonna hate
```

`+=` বাঁ পাশের `var`-এ জোড়ে।

```swift
var title = "Apollo"
title += " 11"
```

`Int`-কে `+` দিয়ে স্ট্রিংয়ে বসানো যায় না। আগে `String` বানাতে হয়।

```swift
let number = 11
// "Apollo " + number + " landed on the moon."   // কম্পাইল এরর
let verbose = "Apollo " + String(number) + " landed on the moon."
```

**চেইন বাঁ থেকে ডানে।** `"1" + "2" + "3" + "4" + "5"` মানে `((( "1" + "2" ) + "3" ) + "4" ) + "5"`। ভাষার মডেলে প্রতি ধাপে নতুন `String`: `"12"`, `"123"`, `"1234"`, শেষে `"12345"`। মাঝেরগুলো শেষ ফলে থাকে না।

সব টুকরা কম্পাইল-টাইম লিটারাল হলে অপটিমাইজার প্রায়ই একটি কনস্ট্যান্টে ভাঁজ করে। ইন্টারভিউতে বলার মতো খরচ লুপের রানটাইম ভ্যালুতে।

```swift
let luggageCode = "1" + "2" + "3" + "4" + "5"
```

`+` ভিতরে বাঁ পাশ কপি করে `append` করে। কলারের ভেরিয়েবল তখনও বাফার ধরে রাখে, তাই সেই বাফার ইউনিক থাকে না। লুপে পুরোনো অংশ বারবার কপি হয়। মোট খরচ টুকরার মোট দৈর্ঘ্যের বর্গের দিকে যায়।

```swift
var slow = ""
for part in parts {
    slow = slow + part          // প্রতিবার নতুন স্ট্রিং
}

var fast = ""
fast.reserveCapacity(total)
for part in parts {
    fast.append(part)           // ইউনিক বাফার জায়গাতেই বাড়ে
}
// fast += part একই রাস্তা। += ভিতরে append।
```

অ্যারে জোড়া লাগাতে `joined`। সেপারেটরসহ দৈর্ঘ্য ধরে একবার বাফার নেয়।

```swift
let lyric = [people, "gonna", action].joined(separator: " ")
```

**ইন্টারপোলেশন।** ব্যাকস্ল্যাশ, তারপর বন্ধনীতে এক্সপ্রেশন। `String`, `Int`, `Double` একইভাবে বসে। ভিতরে অঙ্কও চলে।

```swift
let name = "Taylor"
let age = 26
let message = "Hello, my name is \(name) and I'm \(age) years old."

let missionMessage = "Apollo \(number) landed on the moon."
print("5 x 5 is \(5 * 5)")    // 5 x 5 is 25
```

কোট এড়াতে `\"`। ইন্টারপোলেশনে `\( )`।

```swift
let quote = "Then he tapped a sign saying \"Believe\" and walked away."
```

কম্পাইলার স্ট্রিংটাকে `StringInterpolation`-এ ভাঙে। টুকরাগুলো এক বাফারে `append` হয়। `Int`-এর মতো পরিচিত টাইপে আলাদা `String(number)` না বানিয়ে ডিজিট সরাসরি লেখা হয়। তাই একটি লিটারালে কয়েকটি মান বসানো চেইন করা `+`-এর চেয়ে সস্তা এবং পড়া সহজ।

লুপে `"\(fast)\(part)"` আবার পুরো স্ট্রিং বানায়। বড় জোড়ার জন্য এটা `+`-এর মতোই খরচের জায়গা।

**`\(value)` কী ডাকে।** ডিফল্ট রাস্তা `String(describing:)`। টাইপ `CustomStringConvertible` হলে `description`। না হলে জেনেরিক ডিফল্ট, যা UI-তে দেখানোর যোগ্য নয়। ডিবাগ ডাম্প `String(reflecting:)`, সেখানে `debugDescription`।

এক্সপ্রেশন বাঁ থেকে ডানে, স্ট্রিং বানানোর সময়ই চলে।

```swift
var n = 0
func tick() -> Int { n += 1; return n }
let steps = "\(tick()) \(tick())"   // "1 2"
```

**অপশনাল।** `String?` সরাসরি বসালে লেখা হয় `Optional("Taylor")` বা `nil`। Swift সতর্ক করে। ইউজারকে এই লেখা দেওয়া বাগ।

```swift
let nickname: String? = "Tay"
// "Hi \(nickname)"              // Optional("Tay")
let safe = "Hi \(nickname ?? "there")"
```

`nil` আর খালি স্ট্রিং আলাদা। `??` দিয়ে দুটোরই টেক্সট ঠিক করে দিন।

**র‑ স্ট্রিং।** `#"..."#`-এ `\(name)` ইন্টারপোলেট হয় না। ব্যাকস্ল্যাশসহ ওই অক্ষরগুলোই থাকে। ইন্টারপোলেট করতে `\#(name)`।

```swift
let raw = #"Use \(name) when you want the symbols."#
let cooked = #"Hello \#(name)"#     // Hello Taylor
```

**লোকেল আর SwiftUI।** `"\(1.5)"` ডট রাখে, লোকেল বদলায় না। `String(format: "%f", 1.5)` বর্তমান লোকেল ব্যবহার করে। জার্মান লোকেলে দশমিক কমা হতে পারে। ফাইল নাম, কী, API-তে `Locale(identifier: "en_US_POSIX")` দিন। ইউজারের স্ক্রিনে `NumberFormatter` বা `FormatStyle`।

`Text("Apollo \(number)")` একটি স্ট্রিং লিটারাল, তাই স্ট্রিং ক্যাটালগ এটাকে লোকালাইজেশন কী হিসেবে নিতে পারে। আগে বানিয়ে ফেলা `String` সেই কী নয়।

```swift
Text("Apollo \(number) landed.")          // লিটারাল, লোকালাইজ হতে পারে
Text(missionMessage)                      // তৈরি String, লুকআপ নয়
```

**জোড়ার পর `count` নাও যোগফল হয়।** কম্বিনিং মার্ক আগের ক্যারেক্টারের সাথে মিশে এক grapheme হয়।

```swift
let e = "e"
let accent = "\u{0301}"
e.count                 // 1
accent.count            // 1
(e + accent).count      // 1
(e + accent) == "é"     // true, canonical equivalence
```

`Substring` আর `String` `+` করলে ফল `String`। স্লাইস ধরে রাখার মতো `Substring` থেকে যায় না।

## MCQ

**1.** `let number = 11`। কোনটা কম্পাইল হবে?

A. `"Apollo " + number`  
B. `"Apollo " + String(number)`  
C. `number + " landed"`  
D. `"Apollo " + number + " landed"`  

**Answer: B.** `+` দুই পাশে একই সংখ্যার টাইপ চায়, অথবা দুই পাশে `StringProtocol`। `Int` আর `String` মেশে না। `String(number)` আগে স্ট্রিং বানায়।

**2.** হাজার হাজার টুকরা জোড়ার সবচেয়ে নিরাপদ কোনটা?

A. লুপে `s = s + part`  
B. লুপে `s = "\(s)\(part)"`  
C. `s.reserveCapacity(total)`, তারপর `s.append(part)`  
D. প্রতি টুকরায় `String(format:)`  

**Answer: C.** `append` ইউনিক বাফার জায়গাতেই বাড়ায়। A আর B প্রতি চক্করে পুরো প্রিফিক্স কপি করে। `joined(separator:)` একই শ্রেণির সমাধান।

**3.** `("e" + "\u{0301}").count` কত?

A. `2`, কারণ দুটি `String` যোগ হয়েছে  
B. `1`  
C. `0`, কারণ মার্ক আলাদা ক্যারেক্টার নয়  
D. কম্পাইল এরর  

**Answer: B.** অ্যাকিউট অ্যাকসেন্ট `e`-এর সাথে মিশে একটি extended grapheme। `== "é"` সত্য। `count` বাইট বা যোগফল গোনে না।

**4.** `let nickname: String? = "Tay"`। `"Hi \(nickname)"` কী দেয়?

A. `Hi Tay`  
B. `Hi Optional("Tay")`  
C. `Hi nil`  
D. কম্পাইল এরর, অপশনাল বসানো যায় না  

**Answer: B.** ডিফল্ট ইন্টারপোলেশন `String(describing:)`। অপশনালের ডেসক্রিপশন `Optional("Tay")`। কম্পাইলার সতর্ক করে। ঠিক রূপ `nickname ?? "there"`।

**5.** কোন লিটারালটি `name`-এর মান বসাবে? `name` হলো `"Taylor"`।

A. `#"Hello \(name)"#`  
B. `#"Hello \#(name)"#`  
C. `#"Hello \\(name)"#`  
D. কোনোটাই না। র‑ স্ট্রিংয়ে ইন্টারপোলেশন নেই  

**Answer: B.** র‑ স্ট্রিংয়ে `\( )` সাধারণ অক্ষর। ইন্টারপোলেশনের চিহ্ন `\#( )`। A ছাপে `Hello \(name)`।

**6.** জার্মান লোকেলে কোনটা দশমিক কমা বানাতে পারে?

A. `"\(1.5)"`  
B. `String(format: "%f", 1.5)`  
C. `"5 x 5 is \(5 * 5)"`  
D. `["1", "5"].joined(separator: ".")`  

**Answer: B.** `String(format:)` বর্তমান লোকেল নেয়। ইন্টারপোলেশনের ডিফল্ট সংখ্যা ডট রাখে। মেশিন-কীতে `en_US_POSIX` দিন।

**7.** `Text("Hello, \(name)")` আর `Text(message)` — `message` আগে থেকে বানানো `String`। পার্থক্য কী?

A. দুটোই একই লোকালাইজেশন কী  
B. লিটারালটি ক্যাটালগে কী হতে পারে। তৈরি `String` লুকআপের কী নয়  
C. `Text(message)` কম্পাইল হয় না  
D. ইন্টারপোলেশন SwiftUI-তে নিষিধ  

**Answer: B.** `Text` স্ট্রিং লিটারাল নিলে `LocalizedStringKey`। মান আগে বেক হয়ে গেলে আর `"Hello, %@"` কী থাকে না।

## Short answer

**প্রশ্ন ১. `+`, `+=`, ইন্টারপোলেশন — লুপে কোনটা ব্যবহার করবেন?**  
একটি লিটারালে কয়েকটি মান হলে ইন্টারপোলেশন। কম্পাইলার এক বাফারে টুকরা `append` করে। অ্যারে হলে `joined(separator:)`। লুপে নিজে জোড়ালে `var` নিয়ে `append` বা `+=`, আর দৈর্ঘ্য জানা থাকলে `reserveCapacity`। `s = s + part` আর `s = "\(s)\(part)"` প্রতিবার নতুন স্ট্রিং বানায়, কারণ পুরোনো ভেরিয়েবল বাফার ছাড়ে না। মোট কাজ দৈর্ঘ্যের বর্গের দিকে যায়। শুধু লিটারালের `+` কম্পাইলার ভাঁজ করতে পারে। সেই অপটিমাইজেশন রানটাইম লুপে ধরবেন না।

**প্রশ্ন ২. `+` একই চিহ্ন হয়েও স্ট্রিং জোড়ে আর সংখ্যাও যোগ করে কেন?**  
এটা operator overloading। `+` একটি ফাংশন। কম্পাইলার দুই পাশের টাইপ দেখে কোন ওভারলোড, সেটা বেছে নেয়। `String`-এ জোড়া, `Int` আর `Double`-এ অঙ্ক। টাইপ হয়ে যাওয়া `Int` স্ট্রিং ওভারলোডে বসে না, তাই `"Apollo " + number` কম্পাইল হয় না। `String(number)` বা `"Apollo \(number)"` স্পষ্ট রূপান্তর। নিজের টাইপে `+` তখনই বসাব, যখন যোগ সত্যিই যোগ। অর্থ না মিললে নামওয়ালা মেথড পরিষ্কার।

**প্রশ্ন ৩. ইন্টারপোলেশন ভিতরে কী হয়?**  
`"...\(expr)..."` একটি `String` লিটারাল। কম্পাইলার `DefaultStringInterpolation` বানায়। লিটারালের টেক্সট সরাসরি `append` হয়। এক্সপ্রেশন বাঁ থেকে ডানে চলে। `Int`-এর মতো টাইপে বিশেষ `appendInterpolation` ডিজিট সরাসরি বাফারে লেখে। বাকি টাইপে `String(describing:)`। `CustomStringConvertible` থাকলে `description`। ডিবাগের জন্য `String(reflecting:)`। নিজের ফরম্যাট চাইলে `String.StringInterpolation`-এ `appendInterpolation` যোগ করুন। কল সাইট হয় `\(hex: 255)`।

**প্রশ্ন ৪. অপশনাল স্ট্রিং ইন্টারপোলেশনে কী ভুল হয়?**  
`String(describing:)` অপশনালকে `nil` বা `Optional("Taylor")` লেখে। কম্পাইলার সতর্ক করে, তবু লেখাটি বৈধ এবং UI-তে চলে যেতে পারে। `if let` বা `??` দিয়ে আগে টেক্সট ঠিক করুন। `nil` আর `""` আলাদা অর্থ। দুটোকে এক খালি লেবেলে মেশাবেন না। পোস্টফিক্স `!` দিয়ে আনর‍্যাপ করবেন না। মান না থাকলে সেটা ট্র্যাপ।

**প্রশ্ন ৫. সংখ্যা ইউজারকে দেখানো আর কী বা ফাইলনেম এক রাস্তা কেন নয়?**  
`"\(count)"` লোকেল-নিরপেক্ষ ডিফল্ট ডেসক্রিপশন। `String(format: "%f", value)` বর্তমান লোকেল নেয়। দশমিক কমা, গ্রুপিং সেপারেটর বদলে যায়। API, ফাইলের নাম, ক্যাশে কীতে `en_US_POSIX` দিন। স্ক্রিনে `NumberFormatter` বা `FormatStyle`, ইউজারের লোকেলসহ। SwiftUI-তে `Text("Score: \(score)")` লোকালাইজেশন কী হতে পারে। স্ট্রিং আগে বানিয়ে `Text(message)` দিলে সেই কী আর থাকে না। অনুবাদ লাগলে লিটারালটি `Text`-এর কাছে রাখুন।

**প্রশ্ন ৬. দুটি স্ট্রিংয়ের `count` যোগ করলে ফলের `count` পাওয়া যাবে?**  
সাধারণ অক্ষরে যায়। কম্বিনিং মার্ক আলাদা। `"e"` আর `"\u{0301}"` দুটোর `count` ১, জোড়ার `count`ও ১, কারণ একটি grapheme। Swift সেই জোড়াটাকে `"é"`-এর সমান ধরে। বাইট বা `unicodeScalars.count` দিয়ে UI-এর দৈর্ঘ্য হিসাব করবেন না। `Substring` জোড়ার ফল `String`, তাই বড় বেস বাফার আর ধরে থাকে না।

সিম্পল ডেটার সারাংশ — ইন্টারভিউ নোট (Google Doc-এ কপি করার জন্য)

# 7. Summary: Simple Data

## ৩০ সেকেন্ডে বলার উত্তর

Swift-এর চারটি বেসিক ডেটা `String`, `Int`, `Double`, `Bool`। চারটিই ভ্যালু টাইপ। `let` একবার অ্যাসাইন হয়, `var` পরে বদলায়। মান বদলানোর ইচ্ছে না থাকলে `let`। লিটারাল দেখে টাইপ ঠিক হয়: কোট মানে `String`, ডট মানে `Double`, পূর্ণসংখ্যা মানে `Int`, `true`/`false` মানে `Bool`। একবার টাইপ হয়ে গেলে নিজে থেকে অন্য টাইপে যায় না। `+` টাইপ দেখে আলাদা কাজ করে। টাকায় `Double` ব্যবহার হয় না। `if` শুধু `Bool` নেয়। কয়েকটি মান এক টেক্সটে বসাতে ইন্টারপোলেশন।

## চার টাইপ, এক নজরে

**`let` আর `var`।** `let` বাইন্ডিং একবার পায়। `var` পরে বদলায়। `let` মানে পুরো অবজেক্ট গভীরভাবে অপরিবর্তনীয়, এমন নয়। `let` ক্লাসের `var` প্রপার্টি বদলানো যায়। `let` struct বদলানো যায় না।

**`String`।** ডবল কোট। অনেক লাইনে তিনটি কোট। ইমোজি ও যেকোনো ভাষা grapheme হিসেবে। `count` দৈর্ঘ্য, O(n)। `uppercased()` নতুন স্ট্রিং। `hasPrefix` / `hasSuffix` কেস-সেনসিটিভ `Bool`।

**`Int`।** ধনাত্মক ও ঋণাত্মক পূর্ণসংখ্যা। আজকের iPhone-এ ৬৪-বিট, তবু `Int64`-এর থেকে আলাদা টাইপ। `isMultiple(of:)` গুণিতক চেক করে। overflow-এ ট্র্যাপ। আন্ডারস্কোর শুধু পড়ার সুবিধা: `100_000_000`।

**`Double`।** ডট থাকলে এই টাইপ, `3.0`-ও। IEEE 754। বড় সংখ্যা ধরে, হুবহু দশমিক নয়। `0.1 + 0.2` ছাপে `0.30000000000000004`। টাকা এখানে নয়। সেন্ট `Int`-এ, অথবা স্ট্রিং থেকে `Decimal`।

**অপারেটর।** `+ - * /`। `+= -= *= /=` সরাসরি `var` বদলায়। একই চিহ্ন টাইপ অনুযায়ী ওভারলোড।

**`Bool`।** শুধু `true` বা `false`। `!` নতুন উল্টো মান দেয়। `toggle()` জায়গাতেই উল্টে, তাই `var` লাগে।

**ইন্টারপোলেশন।** `"Hello, \(name). \(5 * 5)"` এক বাফারে মান বসায়। `String` আর `Int` সরাসরি `+` হয় না।

## কোন ফিল্ডে কোন টাইপ

| দরকার | টাইপ |
|---|---|
| নাম, ক্যাপশন, আইডি যেটা টেক্সট | `let String` |
| সংখ্যা, ইনডেক্স, সেন্ট | `Int` |
| রেটিং, স্থানাঙ্ক, অনুপাত | `Double` |
| হ্যাঁ/না জানা | `Bool` |
| এখনো জানা নেই | `Bool?` |
| বদলাবে না | `let` |
| বদলাবে | `var` |

```swift
struct Product {
    let id: String
    let title: String
    let priceCents: Int
    let rating: Double
    var isFavorite: Bool
}
```

গড় রেটিং `Double`। দাম `Int`। দাম `rating`-এর টাইপে রাখা যায় না।

## যে নিয়ম চার টাইপের উপরে বসে

লিটারাল পলিমরফিক। ভেরিয়েবল একবার টাইপ পেলে স্থির।

```swift
let ok = 1 + 2.0          // Double। 1 এখনো টাইপ পায়নি
let a = 1                 // Int
let b = 2.0               // Double
// a + b                  // এরর

var name = "Nicolas Cage"
name = "John Travolta"    // বৈধ
// name = 57              // এরর
```

`if` শুধু `Bool` চায়। `if count` বা `if name` কম্পাইল হয় না।

`&&` আর `||` শর্ট-সার্কিট। ডান পাশ তখনই চলে, যখন বাঁ পাশ যথেষ্ট নয়।

চারটি টাইপই `Sendable` ভ্যালু। কপি সস্তা। একই `var` দুই থ্রেডে লিখলে ডেটা রেস। `+=` অ্যাটমিক নয়।

## মেশানো ফাঁদ

```swift
let filename = "paris.jpg"
let isJPEG = filename.hasSuffix(".jpg")   // Bool। ".JPG" হলে false

var gameOver = false
gameOver.toggle()

let score = 10
let halved = score / 2                    // 5, পূর্ণসংখ্যার ভাগ
var rating = 5.0
rating *= 2                               // 10.0

let message = "Apollo \(score) scored \(rating)."
```

`$0.1 + 0.2 == 0.3` মিথ্যা। `Int.max + 1` ট্র্যাপ। `1.0 / 0.0` হয় `inf`। `Bool?` হলে `if let` `false`-কেও ভিতরে ঢোকায়। শুধু হ্যাঁ ধরতে `== true`। অপশনাল স্ট্রিং `"\(nickname)"` হলে লেখা হয় `Optional("Tay")`। লুপে `s = s + part` বারবার কপি করে। `append` বা `joined(separator:)` এক বাফারে জমা করে।

৬৪-বিট iPhone-এ `CGFloat` আর `Double` একই টাইপ। টাকার অঙ্ক সেখানেও `Double` নয়।

## MCQ

**1.** কোন লাইন `Double` infer করে?

A. `let n = 3`  
B. `let n = 3.0`  
C. `let n = "3.0"`  
D. `let n = true`  

**Answer: B.** ডট থাকলে `Double`, ডটের পরে শুধু `0` হলেও। কোট মানে `String`। `true` মানে `Bool`।

**2.** কোন এক্সপ্রেশন কম্পাইল হবে?

A. `let a = 1; let b = 2.0; a + b`  
B. `let n = 11; "Apollo " + n`  
C. `"5 x 5 is \(5 * 5)"`  
D. `let count = 3; if count { }`  

**Answer: C.** ইন্টারপোলেশনের ভিতরের `5 * 5` দুটোই লিটারাল। টাইপ করা `Int` আর `Double`, বা `String` আর `Int`, নিজে থেকে মেশে না। `if` শুধু `Bool` নেয়।

**3.** কার্টের লাইনে দাম কীভাবে রাখবেন?

A. `var price: Double`  
B. `let priceCents: Int`  
C. `let price: Bool`  
D. `let price: String`, যোগও স্ট্রিং `+` দিয়ে  

**Answer: B.** `Double` দশমিক হুবহু রাখে না। সেন্ট পূর্ণসংখ্যা। স্ক্রিনে পরে ফরম্যাট। স্ট্রিংয়ে যোগ করলে `"19.99" + "0.10"` অঙ্ক হয় না, জোড়া লাগে।

**4.** `let isAdmin: Bool? = false`। কোন ব্লক অ্যাডমিন ধরে ভুল করে ঢোকে?

A. `if isAdmin == true`  
B. `if let isAdmin`  
C. `if isAdmin == false`  
D. কোনোটাই কম্পাইল হয় না  

**Answer: B.** `if let` শুধু `nil` বাদ দেয়। `false` একটি আসল `Bool`, তাই ব্লকে ঢোকে। হ্যাঁ শুধু `== true`-তে।

**5.** `var s = ""`, তারপর হাজার বার জোড়া। কোনটা ঠিক খরচের দিকে?

A. `s = s + part`  
B. `s.append(part)`  
C. `s = "\(s)\(part)"`  
D. `s = s + String(part.count)`  

**Answer: B.** `append` ইউনিক বাফার বাড়ায়। A আর C প্রতি চক্করে নতুন স্ট্রিং বানায়। D সংখ্যাকে টেক্সট বানিয়ে জোড়ে, টুকরা রাখে না।

**6.** কোন সারি মিলে?

A. `Int.max + 1` → `inf`  
B. `1.0 / 0.0` → ট্র্যাপ  
C. `0.1 + 0.2` → হুবহু `0.3`  
D. `120.isMultiple(of: 3)` → `true`  

**Answer: D.** `Int` overflow-এ ট্র্যাপ। `Double` শূন্য দিয়ে ভাগে `inf`। `0.1 + 0.2` বাইনারিতে `0.3` নয়। `isMultiple(of:)` `Bool` দেয়।

## Short answer

**প্রশ্ন ১. সিম্পল ডেটা এক মিনিটে বলুন।**  
চারটি ভ্যালু টাইপ: টেক্সট `String`, পূর্ণসংখ্যা `Int`, ভাসমান `Double`, হ্যাঁ/না `Bool`। বাইন্ডিং `let` হলে একবার, `var` হলে বদলানো যায়। মান না বদলালে `let`, যাতে ভুল অ্যাসাইনমেন্ট কম্পাইলে ধরা পড়ে। লিটারাল দেখে টাইপ infer হয় এবং পরে স্থির থাকে। `+ - * /` আর `+=`-জাতীয় অপারেটর টাইপ অনুযায়ী কাজ করে। স্ট্রিংয়ে মান বসাতে `"\(value)"`। টাকায় `Double` নয়। `Bool` উল্টোতে `!` বা `toggle()`।

**প্রশ্ন ২. এই স্ক্রিনের ডেটা কীভাবে ভাগ করবেন?**  
শিরোনাম ও ছবির নাম `let String`। ছবি JPEG কি না `hasSuffix(".jpg")` দিয়ে `Bool`। পরিমাণ ও দামের সেন্ট `Int`। গড় রেটিং `Double`, সমতা চেক টলারেন্স দিয়ে। ফেভারিট `var Bool`, বোতামে `toggle()`। সার্ভার এখনো জানায়নি এমন ফ্ল্যাগ `Bool?`। `if let` দিয়ে সেটাকে হ্যাঁ ধরবেন না। ইউজারকে এক লাইনে দেখাতে ইন্টারপোলেশন। দামের যোগ সেন্টে `Int` দিয়ে করুন, তারপর ফরম্যাট করুন।

**প্রশ্ন ৩. Swift কোন মিশ্রণ নিজে করে না?**  
টাইপ করা `Int` আর `Double`। `String` আর সংখ্যা। `Bool` আর `0`/`1`। `if`-এর জায়গায় সংখ্যা বা স্ট্রিং। অন্য টাইপ আগে থেকে `String` হওয়া ভেরিয়েবলে। রূপান্তর স্পষ্ট: `Double(a)`, `Int(exactly:)`, `String(number)`, `"\(number)"`। লিটারাল তখনো পলিমরফিক, তাই `1 + 2.0` কম্পাইল হয়। `let a = 1` হওয়ার পর সেই ছাড় থাকে না।

**প্রশ্ন ৪. একই `+` স্ট্রিংয়ে আর সংখ্যায় আলাদা কেন?**  
ওভারলোড টাইপ দেখে বেছে নেয়। `Int`/`Double`-এ অঙ্ক। `String`-এ জোড়া। দুই পাশের টাইপ না মিললে কম্পাইল হয় না, তাই `"Apollo " + 11` ব্যর্থ। সংখ্যা বসাতে `"Apollo \(11)"`। লুপে অনেক টেক্সট হলে `append` বা `joined(separator:)`। `s = s + part` প্রতিবার নতুন স্ট্রিং বানায়।

**প্রশ্ন ৫. এই চার টাইপ নিয়ে প্রোডাকশনে কোন বাগ যায়?**  
দাম `Double`-এ রাখলে এক সেন্টের গরমিল। `0.1 + 0.2 == 0.3` মিথ্যা। `Bool?` নিয়ে `if let` করলে `false`-ও ভিতরে ঢোকে। অপশনাল ইন্টারপোলেশন `Optional("Tay")` ছাপে। পূর্ণসংখ্যার `score / 2` দশমিক কেটে ফেলে। বড় স্ট্রিং লুপে `+` করলে কপি বেড়ে যায়। `Int` overflow ট্র্যাপ করে। `Double` overflow `inf` হয়। দুটোই চেক না করলে ক্র্যাশ বা ভাঙা UI।

অ্যারেতে সাজানো ডেটা — ইন্টারভিউ নোট (Google Doc-এ কপি করার জন্য)

# 8. Arrays (`[Element]`)

## ৩০ সেকেন্ডে বলার উত্তর

`Array` একই টাইপের অনেক মান ক্রমানুসারে রাখে। খালিও হতে পারে। ইনডেক্স `0` থেকে। `[String]` আর `[Int]` আলাদা টাইপ, তাই মেশানো যায় না। এটা struct, তাই কপি অন রাইট। `count` আর সাবস্ক্রিপ্ট O(1)। সীমার বাইরে সাবস্ক্রিপ্ট ট্র্যাপ করে। `append` শেষে যোগ করে। `sorted()` নতুন অ্যারে দেয়। `reversed()` আসল অ্যারে নয়, একটি উল্টো ভিউ। লুপে সামনে থেকে `remove(at:)` করলে পরের ইনডেক্স সরে যায়। ক্লাস এলিমেন্ট কপি করলে রেফারেন্স শেয়ার হয়।

## যে পয়েন্টগুলো বলতে হবে

**এক জায়গায় এক টাইপ, যে ক্রমে বসানো হয়েছে।** ব্র্যাকেট, মাঝে কমা।

```swift
var beatles = ["John", "Paul", "George", "Ringo"]
let numbers = [4, 8, 15, 16, 23, 42]
var temperatures = [25.3, 28.2, 26.4]
```

`beatles[0]` প্রথম। `beatles[1]` দ্বিতীয়। দিন, ছাত্র, বছরের জনসংখ্যা — ক্রম থাকলে অ্যারে।

```swift
print(beatles[0])        // John
print(numbers[1])        // 8
print(temperatures[2])   // 26.4
```

ইনডেক্স না থাকলে অ্যাপ থেমে যায়। `fatalError`, অপশনাল নয়। বৈধ ইনডেক্স `0..<count`। খালি অ্যারেতে `[0]` ট্র্যাপ। `first` আর `last` অপশনাল, না থাকলে `nil`।

```swift
if numbers.indices.contains(1) {
    print(numbers[1])
}
print(numbers.first)     // Optional(4)
```

**টাইপ একবার ঠিক হলে সেই টাইপই থাকে।** `temperatures`-এ `String` ঢোকানো যায় না। বের করলেও যে টাইপ ভিতরে, সেটাই আসে। `[String]` আর `[Int]` দুটোই অ্যারে, তবু আলাদা স্পেশালাইজড টাইপ। `+` তাদের মধ্যে চলে না।

```swift
beatles.append("Adrian")
beatles.append("Allen")
beatles.append("Adrian")   // ডুপ্লিকেট বৈধ। অ্যারে সেট নয়

// temperatures.append("Chris")   // এরর

let firstBeatle = beatles[0]
let firstNumber = numbers[0]
// firstBeatle + firstNumber      // String আর Int মেশে না
```

খালি অ্যারেতে টাইপ লিখতে হয়। নাহলে কম্পাইলার বুঝতে পারে না ভিতরে কী থাকবে।

```swift
var scores = Array<Int>()
var albums = [String]()          // একই জিনিস
scores.append(100)
scores.append(80)
scores.append(85)
print(scores[1])                 // 80

var inferred = ["Folklore"]
inferred.append("Fearless")
inferred.append("Red")
```

মান আগে থেকে থাকলে টাইপ infer হয়। `let` অ্যারেতে `append` বা এলিমেন্ট বদলানো যায় না। `var` লাগে।

```swift
var grid = Array(repeating: 0, count: 100)
```

`repeating` ক্লাসে একই রেফারেন্স বারবার বসায়।

```swift
final class Node { var value = 0 }
let nodes = Array(repeating: Node(), count: 3)
nodes[0].value = 1
// nodes[1].value ও nodes[2].value এখন 1
```

আলাদা অবজেক্ট চাইলে `(0..<3).map { _ in Node() }`।

**চলতি কাজ।**

```swift
print(albums.count)

var characters = ["Lana", "Pam", "Ray", "Sterling"]
characters.remove(at: 2)     // Ray যায়। পরের ইনডেক্স ১ করে সরে
print(characters.count)      // 3
characters.removeAll()
print(characters.count)      // 0

let bondMovies = ["Casino Royale", "Spectre", "No Time To Die"]
bondMovies.contains("Frozen")   // false। == দিয়ে হেঁটে দেখে, কেস-সেনসিটিভ

let cities = ["London", "Tokyo", "Rome", "Budapest"]
print(cities.sorted())
// ["Budapest", "London", "Rome", "Tokyo"]
// cities অপরিবর্তিত

let presidents = ["Bush", "Obama", "Trump", "Biden"]
let reversedPresidents = presidents.reversed()
```

`sorted()` নতুন `[Element]` দেয়। মূল অ্যারে থাকে। `var`-এ জায়গাতেই সাজাতে `sort()`। ডিফল্ট বাড়ানো ক্রম। `String`-এ Unicode কোড পয়েন্ট। বড় হাতের অক্ষর ছোট হাতের আগে আসে। ইউজারের শহরের তালিকায় `localizedStandardCompare`। সমান এলিমেন্টের আগের ক্রম ধরে রাখার গ্যারান্টি নেই। সর্ট স্টেবল নয়।

`reversed()` এলিমেন্ট সরায় না। `ReversedCollection` মনে রাখে যে পড়া উল্টো দিক থেকে। প্লেগ্রাউন্ডে সাধারণ `[String]` দেখায় না। আসল অ্যারে লাগলে `Array(presidents.reversed())`। জায়গাতেই উল্টোতে `reverse()`।

`String`-ও কালেকশন। `"swift".sorted()` দেয় `[Character]`। `String(...)` করলে `"fistw"`।

**খরচ।** `Array`-এর `count` O(1)। `String.count` O(n), কারণ grapheme। এই পার্থক্য ইন্টারভিউতে আসে।

| কাজ | খরচ |
|---|---|
| `array[i]`, `count`, `isEmpty` | O(1) |
| `append`, `popLast` | অ্যামরটাইজড O(1) |
| `insert(_:at:)`, `remove(at:)` | O(n), পেছনের অংশ সরে |
| `contains`, `firstIndex(of:)` | O(n) |
| `sorted()` | O(n log n) |

অনেকবার `append` হলে `reserveCapacity`। লুপে `array = array + [item]` প্রতিবার নতুন বাফারের দিকে যায়। `append` ইউনিক বাফারে জায়গাতেই বাড়ে।

`removeLast()` খালি হলে ট্র্যাপ। `popLast()` খালি হলে `nil`। সামনে থেকে মুছতে মুছতে ইনডেক্স দিয়ে লুপ চালালে এলিমেন্ট এড়িয়ে যান বা ক্র্যাশ। শর্তে মুছতে `removeAll(where:)`।

**কপি অন রাইট।** অ্যাসাইনমেন্টে হেডার কপি হয়, বাফার শেয়ার থাকে। একজন বদলালে আর শেয়ার না থাকলে তখন কপি।

```swift
var a = [1, 2, 3]
var b = a
b.append(4)
// a == [1, 2, 3]
// b == [1, 2, 3, 4]
```

এলিমেন্ট ক্লাস হলে অ্যারের বাফার আলাদা হলেও অবজেক্ট একই থাকে।

```swift
final class Box { var n = 0 }
var first = [Box()]
var second = first
second[0].n = 5
// first[0].n ও 5
```

`let` অ্যারে মানে বাইন্ডিং স্থির। ভিতরের ক্লাসের `var` প্রপার্টি তবু বদলায়।

**স্লাইস।** `numbers[1...3]` একটি `ArraySlice`। নিজের ইনডেক্স মূল অ্যারের ইনডেক্স। এই স্লাইসে `0` নাও থাকতে পারে। বাফার শেয়ার করে, তাই বড় অ্যারের ছোট স্লাইস প্রপার্টিতে রাখলে পুরো বাফার বেঁচে থাকে। আলাদা করতে `Array(slice)`।

`enumerated()` অফসেট সবসময় `0` থেকে। স্লাইসে সেটা মূল ইনডেক্স নয়।

```swift
let slice = Array(numbers.enumerated())[2...]
for (offset, _) in numbers[2...].enumerated() {
    // offset 0, 1, 2... — মূল ইনডেক্স 2, 3, 4 নয়
}
```

**অ্যাপে রাখার নিয়ম।** লোড হয়নি বোঝাতে `nil` অ্যারে। খালি ফল বোঝাতে `[]`। `[Any]` টাইপ সেফটি ফেলে দেয়। ভিন্ন জিনিস এক তালিকায় লাগলে enum। SwiftUI-তে `ForEach`-এর `id` ডেটার পরিচয়। ইনডেক্স দিয়ে `id` দিলে মুছলে বা সর্ট করলে ভিউ ভুল রোতে অ্যানিমেট হয়। ডুপ্লিকেট স্ট্রিংয়ে `id: \.self` দুটি রোকে এক ধরে।

`Codable` অ্যারে JSON অ্যারে হয়। এলিমেন্ট এক টাইপ।

## MCQ

**1.** `var beatles = ["John", "Paul", "George", "Ringo"]`। `beatles[1]` কে?

A. John  
B. Paul  
C. Ringo  
D. কম্পাইল এরর, ইনডেক্স ১ থেকে শুরু  

**Answer: B.** প্রথম উপাদান `0`। `1` দ্বিতীয় উপাদান, Paul। `4` হলে রানটাইমে ট্র্যাপ।

**2.** কোন লাইন কম্পাইল হবে?

A. `var t = [25.3]; t.append("Chris")`  
B. `let a = ["John"]; a.append("Paul")`  
C. `var albums = [String](); albums.append("Red")`  
D. `let s = ["A"]; let n = [1]; s[0] + n[0]`  

**Answer: C.** খালি অ্যারেতে `[String]` স্পষ্ট। `Double` অ্যারেতে `String` বসে না। `let`-এ `append` হয় না। `[String]` আর `[Int]` যোগ হয় না।

**3.** `let cities = ["London", "Tokyo", "Rome", "Budapest"]`। `cities.sorted()` পরে `cities[0]` কী?

A. Budapest  
B. London  
C. কম্পাইল এরর, `let`-এ `sorted` চলে না  
D. Tokyo  

**Answer: B.** `sorted()` নতুন অ্যারে দেয়। `cities` আগের মতোই থাকে, তাই `0` London। জায়গাতেই বদলাতে `var` আর `sort()`।

**4.** `presidents.reversed()`-এর টাইপ কী?

A. `[String]`, উপাদান সরে গেছে  
B. `ReversedCollection<Array<String>>`  
C. `String`  
D. খালি অ্যারে  

**Answer: B.** উল্টো পড়ার ভিউ। সাজানো নতুন অ্যারে চাইলে `Array(presidents.reversed())`। জায়গাতেই উল্টোতে `reverse()`।

**5.** `var a = [1, 2, 3]` আর `var b = a`, তারপর `b.append(4)`। `a.count` কত?

A. 4  
B. 3  
C. কম্পাইল এরর  
D. ট্র্যাপ  

**Answer: B.** অ্যারে ভ্যালু টাইপ। `append` ইউনিক কপি করে। `a` থাকে `[1, 2, 3]`। এলিমেন্ট ক্লাস হলে সেই অবজেক্টের প্রপার্টি দুই অ্যারেতেই বদলাত।

**6.** `Array(repeating: Node(), count: 3)`। `nodes[0].value = 1` হলে বাকি দুটি?

A. `0` থাকে, কারণ তিনটি আলাদা `Node`  
B. `1` হয়, একই রেফারেন্স তিনবার  
C. কম্পাইল এরর  
D. অ্যারে কপি অন রাইটে তিন কপি বানিয়ে ফেলে  

**Answer: B.** `repeating` একই মান তিন বার রাখে। ক্লাস মানে একই অবজেক্ট। আলাদা ইনস্ট্যান্স চাইলে `map` দিয়ে প্রতিবার `Node()`।

**7.** খালি অ্যারে থেকে শেষ উপাদান সরাতে কোনটা নিরাপদ?

A. `removeLast()`  
B. `remove(at: 0)`  
C. `popLast()`  
D. `sorted().removeLast()`  

**Answer: C.** `popLast()` খালি হলে `nil`। `removeLast()` আর `remove(at:)` সীমার বাইরে ট্র্যাপ করে। D-তে `sorted()` নতুন অ্যারে, সেটা `let`-এর মতো টেম্পোরারি, `removeLast` সেখানে বসানো যায় না।

**8.** `"swift".sorted()` কী দেয়?

A. `String` `"fistw"`  
B. `[Character]`, জোড়া লাগালে `"fistw"`  
C. `["swift"]`  
D. কম্পাইল এরর, `sorted` শুধু `Array`-তে  

**Answer: B.** `String` হলো `Character`-এর কালেকশন। `sorted()` অ্যারে দেয়। `String(word.sorted())` হলে `"fistw"`।

## Short answer

**প্রশ্ন ১. অ্যারে কীভাবে একসাথে অনেক ডেটা রাখে, আর টাইপ কীভাবে আটকায়?**  
`Array<Element>` একই `Element`-এর ক্রমযুক্ত তালিকা। লিটারাল `[...]`। খালি হলে `[Int]()` বা `Array<Int>()`। মান দিয়ে শুরু করলে টাইপ infer হয়। ভিতরে দ্বিতীয় টাইপ বসানো যায় না, বের করলেও সেই টাইপই আসে। ডুপ্লিকেট থাকতে পারে। ক্রম অংশ। `[1, 2]` আর `[2, 1]` সমান নয়। ভিন্ন কেস এক তালিকায় লাগলে `[Any]` এড়িয়ে enum রাখুন। না লোড হওয়া আর খালি ফল আলাদা: একটা `nil`, অন্যটা `[]`।

**প্রশ্ন ২. ইনডেক্স আর খরচ কীভাবে বলবেন?**  
ইনডেক্স `Int`, শূন্য থেকে `count - 1`। সাবস্ক্রিপ্ট O(1), কারণ বাফার টানা। `String`-এর ইনডেক্স `Int` নয় এবং `count` O(n)। অ্যারের `count` O(1)। সীমার বাইরে সাবস্ক্রিপ্ট ট্র্যাপ করে। নিরাপদ পড়া `first`, `last`, `firstIndex(of:)`। `append` শেষে অ্যামরটাইজড O(1)। মাঝে `insert` বা `remove(at:)` পেছনের সব উপাদান সরায়, O(n)। `contains` পুরো তালিকা হাঁটে। বারবার খোঁজা লাগলে সেই ডেটা `Set`-এ রাখা যায়, ক্রম দরকার হলে অ্যারেই থাকে।

**প্রশ্ন ৩. `sorted()` আর `reversed()` মূল তালিকা বদলায় না কেন?**  
`sorted()` নতুন অ্যারে বানিয়ে বাড়ানো ক্রমে সাজায়। `let`-এও চলে। জায়গাতেই সাজাতে `var` আর `sort()`। `String`-এ ডিফল্ট সর্ট লোকেল মানে না। বড় হাতের অক্ষর আগে আসে। ইউজারের তালিকায় `localizedStandardCompare`। সমান উপাদানের পুরোনো ক্রম ধরে রাখার গ্যারান্টি নেই। `reversed()` উপাদান সরায় না। `ReversedCollection` উল্টো দিক থেকে পড়ে। `[Element]` লাগলে `Array(...)`। জায়গাতেই উল্টোতে `reverse()`।

**প্রশ্ন ৪. কপি অন রাইট কখন আলাদা কপি দেয়, কখন শেয়ার করে?**  
অ্যারে struct। `var b = a` বাফার শেয়ার করে। `b.append` করলে `b`-এর নিজের বাফার হয়, `a` আগের উপাদান রাখে। এলিমেন্ট ক্লাস হলে দুই অ্যারেতে একই অবজেক্টের রেফারেন্স থাকে। এক পাশের `box.n = 5` অন্য পাশেও দেখা যায়। `Array(repeating: Node(), count: 3)` এক `Node` তিনবার রাখে। `let` অ্যারেতে `append` বা `[i] =` হয় না। ভিতরের ক্লাসের প্রপার্টি বদলানো যায়। কপি অন রাইট লক নয়। এক `var` দুই থ্রেডে লিখলে ডেটা রেস।

**প্রশ্ন ৫. মুছতে মুছতে বা স্লাইস নিয়ে কোন বাগ হয়?**  
`remove(at: 2)` সেই জায়গার উপাদান দেয় এবং পরের সব ইনডেক্স এক ঘর সামনে আনে। সামনে থেকে ইনডেক্স বাড়িয়ে মুছলে পরের উপাদান এড়িয়ে যান। শর্তে `removeAll(where:)`। `popLast()` খালি অ্যারেতে `nil`। `removeLast()` ট্র্যাপ। `array[1...]` একটি `ArraySlice`, ইনডেক্স মূল অ্যারের। প্রপার্টিতে রাখলে বড় বাফার ধরে থাকে। `enumerated()` স্লাইসে `0` থেকে গোনে, মূল ইনডেক্স থেকে নয়। SwiftUI-তে রো-এর `id` ইনডেক্স হলে সর্ট বা ডিলিটে ভুল সেল অ্যানিমেট হয়।

**প্রশ্ন ৬. `String`-এর `sorted()` অ্যারের মতো কী দেয়?**  
`String` হলো `Character`-এর কালেকশন। `"swift".sorted()` একটি `[Character]`: `f`, `i`, `s`, `t`, `w`। আবার স্ট্রিং চাইলে `String(letters)`। ফল `"fistw"`। এটা নতুন কালেকশন। মূল `String` বদলায় না। অক্ষরের ক্রম grapheme অনুযায়ী। কম্বিনিং মার্ক আলাদা করে সাজালে চোখে যা দেখা যায়, সেই দৈর্ঘ্যের সাথে গুলিয়ে ফেলবেন না।

