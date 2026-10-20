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

**Q1. When do you choose `let` over `var`?**  
Use `let` for every binding that is not intentionally mutated. It documents intent, keeps struct values from accidental mutation, and lets the compiler enforce single assignment. Performance gain is usually small. The real win is correctness. I only reach for `var` when the value must change, including `lazy var`, loop variables, and property wrappers such as `@State`.

**Q2. Does `let` make an object immutable?**  
It makes the binding immutable. For a struct, that freezes stored properties, so the value cannot change. For a class, the reference stays fixed and `var` properties can still change. If the struct holds a class, copying the struct copies the reference. Immutability is not deep unless the whole object graph is value types or the class itself exposes no mutation.

**Q3. Why is `lazy var` unsafe to share across threads, while a `static let` is safe?**  
`lazy var` checks and stores the value without a synchronization guarantee, so two threads can both see it as uninitialized and run the initializer. Global and static stored properties are initialized once, lazily, under the runtime’s `once` token. For expensive shared state I use `static let`, or a lock if it truly must be a `lazy var`.

**Q4. What does a capture list copy?**  
`{ [name] in }` evaluates `name` when the closure is created and stores that copy. A `let` or a value type gives the closure an independent value. A class instance copies the reference, so later mutations of the object are still visible. `[weak self]` is the same mechanism used to avoid a retain cycle.

**Q5. How is `inout` different from `var`?**  
A normal parameter is a local constant. To edit a copy I introduce a local `var`. `inout` passes a mutable memory location: the function writes back to the caller’s variable. The caller must pass a `var` with `&`. `inout` is not “pass the object by reference” for classes. Classes are already references. `inout` on a class parameter rebinds the caller’s variable.

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

**Q1. Why is `String.count` O(n), and why is there no integer subscript?**  
Characters are extended grapheme clusters with variable width in UTF-8. The fifth character can start at many different byte offsets, so the string has to walk from the start. `String` is a `BidirectionalCollection`, and it is not random-access. An index from another string is invalid and traps at runtime. To move, use `index(after:)`, `index(_:offsetBy:)`, or `index(_:offsetBy:limitedBy:)`.

**Q2. What is the difference between `String.count` and `NSString.length`?**  
`count` is the number of rendered characters, so a thumbs-up emoji counts as 1. `NSString.length` and `utf16.count` count UTF-16 code units, so that emoji is 2 and a flag can be 4. Mixing `NSRange` from `NSString` with a Swift `String.Index` is a common source of off-by-one bugs when bridging to Foundation or UIKit.

**Q3. How does `String` stay a value type without copying on every assignment?**  
Assignment copies the struct header and retains the same storage. Mutation checks uniqueness. If the buffer has a single owner, `append` grows it in place. If another variable still uses it, the string copies first, then mutates. That is copy-on-write. The same model is used by `Array`, `Dictionary`, and `Set`.

**Q4. When can `uppercased()` change the length, and when does locale matter?**  
Unicode case folding is not one character to one character. German `ß` uppercases to `SS`. Turkish distinguishes dotless `ı` and dotted `İ`, so `uppercased(with:)` can disagree with the default locale. Use locale-aware APIs for user-facing search and sort. For identifiers, file extensions, and tokens, compare a locale-independent lowercase so results do not change with the device language.

**Q5. Why should a stored property be `String` and not `Substring`?**  
Slicing with `prefix`, `suffix`, or a range returns a `Substring` that shares the original buffer. Holding that slice keeps the entire original allocation alive. `String(sub)` detaches storage so the large source can be released. Slicing inside a function is fine. Store, return from your API, or use as a dictionary key only after turning it into a `String`.

**Q6. How do you build a large string efficiently?**  
Use a `var` and `append`, and call `reserveCapacity` when the size is known. Repeated `text = text + chunk` can copy the growing buffer on each iteration. Interpolation (`"\(a) \(b)"`) is the normal choice for a fixed number of pieces. For many pieces, append into one mutable string.

পুরো সংখ্যা (`Int`) — ইন্টারভিউ নোট (Google Doc-এ কপি করার জন্য)

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

**Q1. Why does Swift trap on integer overflow instead of wrapping?**  
A wrong wrap becomes a later logic bug or a security bug. Swift’s default operators check the result and trap in debug and release, so the failure stays next to the bad arithmetic. When wrapping is the real spec, as in hashing or bit masks, use `&+`, `&-`, and `&*`. When the caller must decide, use `addingReportingOverflow` and read the overflow flag.

**Q2. What is the difference between `%` and a modulo operation?**  
`%` is the remainder of division toward zero, and it keeps the sign of the left operand. `-9 % 2` is `-1`. Mathematical modulo for a positive divisor is never negative. Also, `Int.min % -1` traps because the quotient does not fit in `Int`. For a multiple check I use `isMultiple(of:)`. It returns `false` for a nonzero value divided by `0`, `true` for `0.isMultiple(of: 0)`, and it does not trap on `Int.min`.

**Q3. Why do API guidelines prefer `Int` when the value cannot be negative?**  
`UInt` still traps on underflow, and it does not model “must be positive” in the type system in a useful way. `Int` and `UInt` do not convert implicitly, so mixing them spreads casts through the code. Counts, indices, and `Array.count` are `Int`. I use a fixed-width or unsigned type for a C struct, a file format, or an actual bit width such as `UInt8` for bytes.

**Q4. How do you convert numbers without surprising truncation?**  
`Double(int)` is exact for integers up to the 53-bit mantissa. `Int(double)` truncates toward zero, so `Int(-2.9)` is `-2`. `Int(exactly:)` returns `nil` when the value is not an integer or does not fit. The same split exists for widths: `Int8(someInt)` traps if the runtime value is out of range, and `Int8(exactly:)` returns an optional. A literal that does not fit, such as `let x: Int8 = 1000`, is a compile error.

**Q5. `Int` and `Int64` are the same size on iPhone. Why can they not be used interchangeably?**  
Same size is a platform fact. The types are still different, and Swift has no implicit numeric conversion. On a 64-bit device `MemoryLayout<Int>.size` and `MemoryLayout<Int64>.size` are both 8, yet `let x: Int64 = someInt` does not compile. `Int` is pointer-sized, so its width follows the platform. `Int64` stays 64-bit everywhere. Convert at the boundary with `Int64(someInt)` or `Int(exactly: someInt64)`.

**Q6. How would you parse a number a user typed?**  
`Int(text)` accepts an optional sign and a radix, and it returns `nil` for `"42a"` or for surrounding spaces. It does not understand grouping separators or Bengali digits. For those I use `NumberFormatter` with the user’s locale. I keep the result optional until validation succeeds. I do not use `0` as a stand-in for “missing”.
