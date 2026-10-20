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

---

পরের টপিক এই একই ছকে দেব: ৩০ সেকেন্ডের উত্তর, ইন্টারভিউ পয়েন্ট, কোড, MCQ, তারপর short answer।

- [Understanding Custom UIView In-depth: Setting File Owner vs custom class](#understanding-custom-uiview-in-depth-setting-file-owner-vs-custom-class)

## Understanding Custom UIView In-depth: Setting File Owner vs custom class
Before we jump into creating custom views, let’s see `two different ways` to load XIB File:

1. Using the `UINib` method instantiate(withOwner:options:)

```swift
let nib = UINib(nibName: "nibFileName", bundle: nil)
if let view = nib.instantiate(withOwner: self, options: nil).first as? UIView {
  // view is ready
}
```

2. Using the `Bundle` method loadNibNamed(_:owner:options:)

```swift
if let view = Bundle.main.loadNibNamed(“nibFileName”, owner: self, options: nil)?.first as? UIView {
  // view is ready
}
```

Tip 1: If you notice well, both the methods are returning an array of views, therefore I had to use `.first` to get the view instance.

Can you guess why? (Think before scrolling further 😉)

> XIB can have multiple views. So return value is an array containing the top-level objects in the nib file. As you can see in the screenshot:

![image](https://user-images.githubusercontent.com/40492085/224464885-f1049616-e608-43e4-82e2-bb2f0d0d2e2b.png)

I would still recommend to use only one view in one XIB, it’s a cleaner approach to avoid confusion later.

Now we are ready to create custom UIView classes. But, it seems like a lot of developers have different opinions on being asked what’s the best way to create a custom UIView class. Mostly, the confusion is around Setting File owner and setting custom class in XIB

The best way to find an answer is to try out different approaches and find out ourself 💪

We are going to create custom UIView using `3 different experiments`:

[A. Set File owner.](#experiment-a-create-a-custom-view-using-file-owner)

[B. Set Custom class.](#experiment-b-create-a-custom-view-using-custom-class)

[C. Set both File owner and custom class ?](#experiment-c-create-a-custom-view-using-custom-class-and-file-owner-both)

#### Setting up the project
Let’s create a TabBar application so that we can use different tab bars for every experiment. [Using XCode 11]

![image](https://user-images.githubusercontent.com/40492085/224465047-0043c64c-d2d0-4cce-a507-38f9693c7da8.png)

Setting up different controllers for each experiment

![image](https://user-images.githubusercontent.com/40492085/224465064-ac9c0f7e-e9de-451d-98fd-3a05c47c7776.png)

The best way to find an answer is to try out different approaches and find out ourself 💪

Now we are ready with a Tab Bar Application which has 3 controllers ExperimentA_controller, ExperimentB_controller, ExperimentC_controller each represents its own experiment

### Experiment A: Create a custom view using `File owner`
**Step 1:** Add a new class `ChangeColorExpA.swift`

![image](https://user-images.githubusercontent.com/40492085/224465085-5a04b79e-b721-4739-be3a-24a8c2de3da1.png)

**Step 2:** Add a new view `ChangeColorExpA.xib`

![image](https://user-images.githubusercontent.com/40492085/224465107-49754744-192a-4f65-94a5-06b7d00af779.png)

**Step 3:** Resize View (Optional Step) using Attribute Inspector and add a button at the center of View as shown below:

![image](https://user-images.githubusercontent.com/40492085/224465154-9048712e-9367-4eb4-a33c-71fc7dff5ad0.png)

**Step 4:** Set File owner of class

![image](https://user-images.githubusercontent.com/40492085/224465177-119486b2-733e-4a93-a41b-6751b7f95461.png)

**Step 5:** Load XIB through code. Go to ChangeColorExpA.swift and use the code snippet as mentioned below

```swift
class ChangeColorExpA: UIView {
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        customInit()
    }

    required init?(coder: NSCoder) {
        super.init(coder: coder)
        customInit()
    }
    
    private func customInit() {
        let nib = UINib(nibName: "ChangeColorExpA", bundle: nil)
        if let view = nib.instantiate(withOwner: self, options: nil).first as? UIView {
            view.backgroundColor = UIColor.clear
            addSubview(view)
            view.frame = self.bounds
            self.backgroundColor = UIColor.cyan
        }
    }
    
    @IBAction func changeColorButtonTapped(_ sender: Any) {
        let colors: [UIColor] = [UIColor.red, UIColor.green, UIColor.yellow, UIColor.orange, UIColor.black, UIColor.purple, UIColor.blue]
        let randomColorIndex = arc4random() % 6
        self.backgroundColor = colors[Int(randomColorIndex)]
    }
}
```

Hurray! 😀 ‘ChangeColorExpA’ View is ready for use now

**Tip 2:** We set the file owner in xib already, then why we have to set it again in the code snippet as mentioned above.

Can you guess why? (Think again before scrolling further 😉)

Setting file owner in xib only sets the Type of class owning the xib [not instance of owner] which later allows you to connect outlets and IBActions to the corresponding owner.

You still need to set owner [instance of owner this time] through code while loading Xib.

**‘ChangeColorExpA’** View is finally ready to use, but how?

**Step 6:** Go to ExperimentA_Controller in the storyboard and change the class of view.

Run the app.

![image](https://user-images.githubusercontent.com/40492085/224465848-4dfe75d4-fbbc-418a-9166-964538b8de1e.png)

As you can see, `ChangeColorExpA` is loaded successfully.

**Tip 3:** How did this get loaded, even though we are not calling any of the init methods? Guess Why?

> As soon as system loads ViewController:
>
> System calls init(coder: NSCoder) of our custom view as it tries to unarchive the nib file because we have set custom class (ChangeColorExpA) to empty view in the storyboard. As a result, our method customInit() gets called where we are loading our xib file.

### Experiment B: Create a custom view using `Custom Class`

**Step 1:** Add new files: `ChangeColorExpB.swift` and `ChangeColorExpB.xib` similar to `experiment A`

**Step 2:** Resize View (Optional) using Attribute Inspector and add a button at the center of View similar to `experiment A`

**Step 3:** Set Custom Class this time instead of File Owner, see below:

![image](https://user-images.githubusercontent.com/40492085/224466149-f41a0b38-1e77-44bc-995f-58e33458683d.png)

**Step 4:** Load XIB through code. Go to `ChangeColorExpB.swift` and use the code snippet similar to `Experiment A` (just use correct xib name while loading it)

**Step 5:** Go to `ExperimentB_Controller` in the storyboard and change the class of empty view. (similar to experiment A)

Let’s see what happens when we run and go to `Experiment B` tab bar item?

![image](https://user-images.githubusercontent.com/40492085/224466266-c4c402e0-9647-4877-9687-8876545d2148.png)

Oops! Did something go wrong? Unfortunately, this doesn’t work fine

Is App hanging? Can you guess why? 🤨 (Hint: Check Tip 3 in Experiment A)

> As soon as system loads `ViewControllerB`:
>
> System calls init(coder: NSCoder) of our custom view, (same as experiment A) which will call our method customInit().
>
> Inside customInit() we are loading our xib file, and in our xib file we have set custom class to ChangeColorExpB (Done in Step 3). As a result, system will call init(coder: NSCoder) again for ChangeColorExpB. Since, it’s a never ending loop if you notice, your App will hang. 😥

Then, what should be the correct way to create a custom view without using File Owner? 🙄

1. Don’t load xib with in the same class in any of the init methods (init(coder: NSCoder) or init(frame: CGRect)) to avoid an infinite loop
2. We have to load xib in the parent class or somewhere else instead of a custom UIView class.

**Step 6:** Load xib in parent controller class i.e. `ExperimentB_controller`

Check the code snippet for parent class below:

```swift
class ExperimentB_controller: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        addChangeColorView()
    }
    
    func addChangeColorView() {
        if let firstView = Bundle.main.loadNibNamed("ChangeColorVarB", owner: nil, options: nil)?.first as? UIView {
            view.addSubview(firstView)
            firstView.frame = CGRect(x: 0, y: 100, width: self.view.frame.size.width, height: firstView.frame.size.height)
            firstView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
            firstView.backgroundColor = UIColor.cyan
        }
    }
}
```

Your custom view `ChangeColorVarB` class will only have `outlets` and `actions`:

```swift
class ChangeColorVarB: UIView {
    
    @IBOutlet var changeColorButtonInCustomView: UIButton!
    
    @IBAction func changeColorButtonTappedInCustomView(_ sender: Any) {
        let colors: [UIColor] = [UIColor.red, UIColor.green, UIColor.yellow, UIColor.orange, UIColor.black, UIColor.purple, UIColor.blue]
        let randomColorIndex = arc4random() % 6
        self.backgroundColor = colors[Int(randomColorIndex)]
    }
}
```

As you might have noticed, I have set the **owner to nil** in code.
**Tip:** Can you guess what will happen if you change the owner to self i.e. parent class in this case?

> If you assign parent class as an owner to our custom view [both in xib and code], It works perfectly fine. You will be able to connect outlets to the parent class too. Also, at the same time, you would still be able to connect outlets in the custom view.
>
> Check experiment C to go more deep:

### Experiment C: Create a custom view using `Custom Class` and `File Owner` both
**Step 1:** Add new files: `ChangeColorExpC.swift` and `ChangeColorExpC.xib` similar to `experiment A`

**Step 2:** Add a button at the center of View similar to `experiment A and B`

**Step 3:** This time, we are going to set both custom class [to custom view class] and file owner [to parent class]. Check in the screenshots:

![image](https://user-images.githubusercontent.com/40492085/224466755-40bc0f0d-3693-4fbf-ba9e-79cc08a848a3.png)

**Step 4:** Load xib in parent class. Connect `outlets` and `IBAction` to both parent and custom view class.

> Why are we loading xib in parent, not in custom view class ?
>
> Learnings from experiment B - As we have given custom class to xib already, we should not load xib in same class to avoid inifinte loops.

![image](https://user-images.githubusercontent.com/40492085/224466787-bd497cd4-79bf-4582-bf1e-36a1c7db7790.png)

Let’s run and observe what’s happening here:

![image](https://user-images.githubusercontent.com/40492085/224466802-a991ff2c-0ff4-4285-a14a-8961b17110b0.png)

As you can see both the outlets are exactly the same object.

So after playing around with views, file owners and custom class. We can easily say Experiment A wins here. 🥳 Can you guess why?

Pros of using Experiment A [Setting only ‘File Owner’ in xib]:

> We can reuse custom view throughout the app using code, storyboards, and xib. Other ways can have a never-ending loop as we saw already.
>
> No hassle to load xib outside from parent class and other classes.
>
> Cleaner approach, as File owner is itself class only instead of a parent or some other class.
>

Overall, this approach gives us a reusable class which is packed in itself to avoid confusions of having different file owner or multiple outlets.

We can make a slight improvement in this approach by connecting the root view of our custom view xib to our class.

![image](https://user-images.githubusercontent.com/40492085/224466953-89e9fb2b-dfb8-416f-81c8-3990e7c018ac.png)

How is it better? 😎
In case, you want to add/update any views or update constraints it’s better to have an outlet of root view. As contentView outlet will contain all the subviews of xib.

you can download the complete project from [**Github**](https://github.com/bhupendratrivedi/SampleCustomView) and play around with it.
