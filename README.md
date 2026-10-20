# Swift Documentation

![image](https://user-images.githubusercontent.com/40492085/224463912-b3352fab-c1ad-4a09-8a7e-08c13d9dbf20.png)

# ভেরিয়েবল ও কনস্ট্যান্ট তৈরি করা

প্রোগ্রামে ডেটা রাখতে হয়। যেমন: ইউজারের নাম, ইন্টারনেট থেকে আনা খবর, বা কোনো হিসাবের ফল। Swift-এ ডেটা রাখার দুটি উপায় আছে। পার্থক্য একটাই: মান পরে বদলাতে পারবেন কি না।

- `var` — ভেরিয়েবল। মান পরে বদলানো যায়।
- `let` — কনস্ট্যান্ট। মান একবার দিলে আর বদলানো যায় না।

নতুন Xcode Playground খুললে সাধারণত এই লাইন থাকে:

```swift
var greeting = "Hello, playground"
```

এখানে `greeting` নামের একটি ভেরিয়েবল তৈরি হয়েছে। এটা ভেরিয়েবল, তাই প্রোগ্রাম চলার সময় এর মান বদলানো যাবে।

টিপ: macOS Playground-এ আরেকটি লাইন থাকে — `import Cocoa`। এটা Apple-এর তৈরি বিশাল কোড লাইব্রেরি নিয়ে আসে। এটা মুছে ফেলবেন না।

## এই এক লাইনে চারটি অংশ

```swift
var greeting = "Hello, playground"
```

1. `var` — নতুন ভেরিয়েবল তৈরি করার কীওয়ার্ড।
2. `greeting` — ভেরিয়েবলের নাম। যেকোনো নাম দিতে পারেন, তবে নাম দেখে বোঝা যাওয়া ভালো।
3. `=` — এই চিহ্ন দিয়ে মান বসানো হয়। দুই পাশে স্পেস না দিলেও চলে, তবে স্পেস রাখাই সাধারণ স্টাইল।
4. `"Hello, playground"` — যে মান বসানো হচ্ছে। টেক্সট সবসময় ডবল কোট (`"`) এর ভিতরে লিখতে হয়, যাতে Swift বুঝতে পারে লেখা কোথায় শুরু ও শেষ।

Swift-এ লাইনের শেষে সেমিকোলন (`;`) লাগে না। সেমিকোলন চলে, কিন্তু প্রায় ব্যবহার হয় না। একই লাইনে দুটি কোড লিখতে চাইলে তবেই দরকার হয়।

## ভেরিয়েবলের মান বদলানো

```swift
var name = "Ted"
name = "Rebecca"
name = "Keeley"
```

ধাপ:

1. `var name = "Ted"` — `name` নামের ভেরিয়েবল তৈরি হলো, মান `"Ted"`।
2. `name = "Rebecca"` — আগের মান বদলে `"Rebecca"` হলো।
3. `name = "Keeley"` — আবার বদলে `"Keeley"` হলো।

দ্বিতীয় ও তৃতীয় লাইনে `var` নেই, কারণ নতুন ভেরিয়েবল তৈরি হচ্ছে না। যেটা আগেই আছে, শুধু তার মান বদলানো হচ্ছে। যতবার খুশি বদলাতে পারেন। নতুন মান বসালে পুরোনো মান ফেলে দেওয়া হয়।

## কনস্ট্যান্ট — যে মান বদলায় না

মান কখনো বদলাবে না জানলে `let` ব্যবহার করুন। লেখা প্রায় `var`-এর মতোই, শুধু কীওয়ার্ড আলাদা।

```swift
let character = "Daphne"
```

এখন `character` একটি কনস্ট্যান্ট। এর মান বদলানোর চেষ্টা করলে Xcode বড় এরর দেখাবে।

এই কোড চলবে না:

```swift
let character = "Daphne"
character = "Eloise"      // এরর
character = "Francesca"   // এরর
```

এখানেও দ্বিতীয় ও তৃতীয় লাইনে `let` নেই, কারণ নতুন কনস্ট্যান্ট তৈরি হচ্ছে না। আগেরটাকে বদলানোর চেষ্টা হচ্ছে, আর সেটাই নিষিধ। কনস্ট্যান্ট বদলালে আর কনস্ট্যান্ট থাকে না।

`let` শব্দটা গণিত থেকে এসেছে। গণিতে বলা হয়, “let x be equal to 5”, অর্থাৎ ধরে নিন x এর মান 5।

গুরুত্বপূর্ণ: যে দুটি লাইনে এরর আসছে, সেগুলো মুছে দিন। কনস্ট্যান্ট বদলানো সম্ভব নয়।

## মান প্রিন্ট করে দেখা

শেখার সময় যেকোনো ভেরিয়েবলের মান `print` দিয়ে দেখা যায়। আসল অ্যাপে ইউজার এই প্রিন্ট দেখে না, তাই সেখানে কম কাজে লাগে। কিন্তু ডেটার ভিতর কী আছে, তা বোঝার সহজ উপায়।

```swift
var playerName = "Roy"
print(playerName)

playerName = "Dani"
print(playerName)

playerName = "Sam"
print(playerName)
```

যা হবে:

1. `playerName` তৈরি হলো, মান `"Roy"`। প্রিন্ট: `Roy`
2. মান বদলে `"Dani"`। প্রিন্ট: `Dani`
3. মান বদলে `"Sam"`। প্রিন্ট: `Sam`

Playground-এ কোড চালাতে বাঁ দিকের নীল Play আইকনে ক্লিক করুন। নীল স্ট্রিপের উপরে-নিচে সরালে আইকনও সরে। এতে নির্দিষ্ট লাইন পর্যন্ত কোড চালানো যায়। বেশিরভাগ সময় শেষ লাইন পর্যন্ত চালালেই হয়।

## নামকরণ — camelCase

Swift নাম নিয়ে কড়া নয়। `playerName`, `playername`, `player_name` — সবই চলবে। কিন্তু একবার যে নাম দিয়েছেন, পরে হুবহু সেই নামই ব্যবহার করতে হবে। `playerName` আর `playername` Swift-এর কাছে দুটি আলাদা নাম।

Swift ডেভেলপারদের সাধারণ নিয়ম camelCase:

- প্রথম শব্দ ছোট হাতের অক্ষরে।
- পরের প্রতিটি শব্দ বড় হাতের অক্ষর দিয়ে শুরু।

```swift
let managerName = "Michael Scott"
let dogBreed = "Samoyed"
let meaningOfLife = "How many roads must a man walk down?"
```

## মনে রাখার নিয়ম

যেখানে মান বদলানোর দরকার নেই, সেখানে `var` না দিয়ে `let` দিন। এতে Swift কোড একটু ভালোভাবে অপটিমাইজ করতে পারে, আর ভুল করে মান বদলে ফেলাও আটকে যায়।



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
