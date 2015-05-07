---
layout: default
---

# My first „Hello World!” processor!

This guide will show you how to create the new processor of yours, add some layers to it and set their properties.

*A bit of disclaimer: this guide will continuously evolve as developer documentations do. But hey, it exists! So it worth to come back from time to time or subscribe to its change-notification. The latest state of this text always reflects the HEAD version of branch „szotsaki”. Generally speaking, if something doesn't work for you, just merge and recompile everything.*

## So, the basics
You want to write a processor which does some calculations. You also want to show your work and later maybe interact with it on the GUI. For all of these you are needed to deal with three classes:

1. `Processor`: of course this is the most important one. All the essence of your work goes mostly here.
2. `Layer`: if you want some show-off, you'll also need this. Or these. You have the possibility to attach more layers to a processor. Or none. It's truly your choice whether you want to show something to the world.
3. `ProcessorSlot`: last but not least, a structure is needed which encompasses these aforementioned classes together. You won't need to care about this much though, its defaults are just perfect.

### Processors
First, start with the `Processor`. It has a *name* and arbitrary number of custom *properties*. The processor-related files are stored with the model, in the <tt>MarbeCommon/Processors</tt> directory. To create your first processor, let's call it `FirstProcessor`, you have to subclass from the abstract `Processor` class like this:

```c++
class FirstProcessor : public Processor
{
    Q_OBJECT
public:
    FirstProcessor(QObject *parent = nullptr);
    ~FirstProcessor() = default;
}
```

The `Q_OBJECT` macro is necessary because of some boring administration reasons. The `parent` pointer is necessary because [Qt](http://www.qt.io/) maintains a hierarchy of the declared objects for easier deletion among others. After that you see the constructor and destructor. Here is, how to define this class:

```c++
FirstProcessor::FirstProcessor(QObject *parent)
    : Processor("My first Processor", parent)
{
}
```

We pass the name of our processor (which will be shown on the GUI) to the parent constructor.

### Layers
Now, head towards to the `Layers`.

The resemblance is uncanny with the processors. You will just have to create a new `Layer` class of yours (call it, for the sake of simplicity, `FirstLayer`), make it a descendant of the `Layer` abstract class and pass your *layer's name* to the parent constructor. The layer-related files lie with other GUI files, in the <tt>GrainAutLine/Layers</tt> directory. A very basic implementation would look like this:

```c++
class FirstLayer : public Layer
{
    Q_OBJECT
public:
    explicit FirstLayer(QObject *parent = nullptr);
    ~FirstLayer() = default;
}
```

```c++
FirstLayer::FirstLayer(QObject *parent)
    : Layer("My first layer", parent)
{
}
```

### The ProcessorSlot
Let's finish with the encapsulating class, the `ProcessorSlot`.

First, you need to shortly name your work you did so far. The name of the compilation will appear only in code but it should be expressive for other developers. Register this name in the `ProcessorSlot::Type` enumeration. Now you can create a new specific instance of `ProcessorSlot`: please, usher yourself to the hall of the `ProcessorSlot` constructor and supplement the switch-case structure with the name of your choice.

The `Processor_` variable will hold your custom processor. Create a new instance of it:
```c++
Processor_ = std::make_shared<FirstProcessor>(this);
```
The `Layers` vector holds all the layers you want to use later. Previously we created only one, `FirstLayer`, but feel free to either add it multiple times or create many different layers later and add them here:
```c++
Layers.push_back(std::make_shared<FirstLayer>(this));
```

Note that layers are optional extensions for a `Processor` determined to show the internal state of the latter. Therefore it's not necessary to attach a layer at all.

### The almighty encapsulator: the SlotManager
Maybe there's a slight chance you've been wondering if there is an encapsulation class for `Layers` and `Processors` (this is the `ProcessorSlot`), is there a similar container for many `ProcessorSlots`? Yes, there is. It is called `SlotManager` which, not surprisingly, manages the life cycles of `ProcessorSlots`.

When you are ready with your modifications so far, you are needed to register your `ProcessorSlot` in `SlotManager`. This is quite easy, just add your class to the following container in its constructor like this:
```c++
NonElementarySlots.push_back(
    std::make_shared<ProcessorSlot>(ProcessorSlot::Type::MyFirstProcessorSlot, this)
  );
```

Make sure that you replace `MyFirstProcessorSlot` with the name you used previously in the enumeration.

### Finishing

You created a new  `Processor` and a new `Layer`. Then you added them to the their encapsulation class, `ProcessorSlot`. After that you registered this processor slot at their manager, `SlotManager`.

Now, we are ready. Oh, just one thing: make sure that your processor computes and layer draws. How easy to say that, isn't it? Well, it will surely be a joyful journey. For the beginning, define and override the following functions:
- for the computation itself:
```c++
virtual bool FirstProcessor::Step(ProcessingStateDescriptor& psd) override;
```
- and for the drawing:
```c++
virtual void FirstLayer::Render(const ProcessingStateDescriptor& psd) override;
```

Have a nice time coding!

# Reference documentation

From now on, we are going through the different trifles of the API to make you get most out of the programming interface.

## ProcessorSlot
As we saw previously, a `ProcessorSlot` encompasses exactly one `Processor` and an arbitrary number of `Layers`.

### Elementary ProcessorSlots
An *elementary* `ProcessorSlot` has the following properties:
- Its `Processor` always runs regardless of everything. This is necessary for making such fundamental tasks done like the raw image rendering.
- Its `Processor` name doesn't show up in the drop-down processor selector list on the GUI, so it cannot be selected to run alone.
- This infers that an elementary `Processor` cannot have properties configured from the GUI, since its property window will never be displayed.
- Nevertheless, it still can have as many layers as you want and these layers are always shown until the user decides to turn them off.

To make a `ProcessorSlot` elementary, just set the `Elementary_` variable to `true` in the `ProcessorSlot` constructor. Just like the following example does:

```c++
[…]
case Type::ShowImage:
        Processor_ = std::make_shared<ShowImageProcessor>(this);
        Layers.push_back(std::make_shared<ShowImageLayer>(this));
        Elementary_ = true;
        break;
[…]
```
This property cannot be modified at run-time.

Please, pay attention to instantiate your elementary `ProcessorSlot` in the right section of the `SlotManager` constructor. A comment line indicates where these are going.

### Supplementary canvas

A *supplementary canvas* is a small area displaying custom layers which is bound to a `ProcessorSlot` and shown on the right hand side of the GUI.

This feature is under construction; please check back a while later for documentation.

## Processor

### Processor properties

A processor is a rather complex entity, therefore it needs some *properties* to fine-tune itself. These properties are shown on the GUI and freely configurable by the user. All properties have a unique *name*, a *default value* and of course they hold their *current value*.

To add a property to your `Processor`, simply call the `AddProperty` method in your `Processor` constructor.

Currently, there are two different kinds of properties you can use:

1. *Float type*: You can store a `float` value inside with a *minimum* and *maximum* range provided.
    ```c++
    void AddProperty(const QString& name,
                     const float minimum,
                     const float maximum,
                     const float default_);
    ```
    Reading its value is possible by its name with the following function:
    ```c++
    float GetFloatPropertyValue(const QString& name);
    ```

2. *Boolean type*: You can store a `bool` value inside.
    ```c++
    void AddProperty(const QString& name,
                     const bool default_);
    ```
    Reading its value is possible by its name with the following function:
    ```c++
    bool GetBoolPropertyValue(const QString& name);
    ```


### Overriding Run()
A `Processor` works this simplified way: when a mouse click occurred or a new image file was loaded, the `Run` method starts and calls `Step` until it returns `false`. If you need more elaborate control, you can override this method. Its default implementation looks like the following:

```c++
void Processor::Run(ProcessingStateDescriptor& psd)
{
    while(Step(psd) == true)
        ;

    emit ComputationReady(psd);
}
```

Please, always emit the `ComputationReady` signal when the `Run` process finishes.

### Mouse handling

#### Subscribing to the mouse events

There are four types of mouse events exist:

| Constant                      | Description       |
|-------------------------------|-------------------|
| `QEvent::MouseButtonPress`    | Mouse press       |
| `QEvent::MouseButtonRelease`  | Mouse release     |
| `QEvent::MouseButtonDblClick` | Mouse press again |
| `QEvent::MouseMove`           | Mouse move        |

The `MouseButtonRelease` is e.g. useful for one-shot actions like running the processor after a click was made at a certain coordinate. While the `MouseMove` is best for continuous actions like drawing. You can freely subscribe to each of them independently with your processor.

By default, a `Processor` doesn't receive these mouse events. To do so, add the desired ones to the `SubscribedMouseEvents` [`unordered_set`][unordered_set] structure like this:

```c++
SubscribedMouseEvents = {QEvent::MouseButtonRelease, QEvent::MouseMove};
```

You may add them directly at the construction time of your class, although you can modify these values any time you want; the changes will come into effect before the next run of your `Processor`.

Please note that regardless you are subscribed to the mouse events, your `Processor` won't receive them if it's not the currently selected one.

#### Reacting to a mouse event

Supposing you're already subscribed to the desired mouse events, your processor's will receive them in the following `virtual` function:

```c++
virtual void MouseEventOccurred(const QMouseEvent& event,
                                const QString& canvasId,
                                ProcessingStateDescriptor& psd)
```

The first variable contains the event itself. You can determine its type by using the `type()` function of its. For example if you are subscribed to the previously mentioned events, you can distinguish between them in the following way:

```c++
void MyFirstProcessor::MouseEventOccurred(const QMouseEvent &event, const QString &canvasId, ProcessingStateDescriptor &psd)
{
    // Refresh the MouseEvent variable both on QEvent::MouseButtonRelease and QEvent::MouseMove
    MouseEvent = event;

    if (event.type() == QEvent::MouseButtonRelease) {
        MouseClicked(canvasId, psd);
    }
}
```

The default implementation of this function stores the `QMouseEvent` object into the `MouseEvent` member variable. In case you don't need extra handling of the events, you may not want to override this function, just read the `MouseEvent` member from your `Processor`.

#### Getting the current mouse coordinates

When a mouse event occurred and the processor is set to receive these events, by default the `MouseEvent` member contains the current event with its position. Use the `pos` or `localPos` functions to read out these coordinates. Please, refer to the [QMouseEvent][QMouseEvent] and [QPoint][QPoint] documentation on handling these values.

Please note that because of performance reasons functions `screenPos`, `windowPos` and `globalPos` always return with `QPoint(0, 0)`.

The top-left image coordinate is (0, 0) and the given coordinates are always resize-agnostics.

## Layers

A `Layer` renders a bit of portion of the main image which will be blended into the renderings of other layers. After all the layers created their own `cv::Mat` image, the so called `ImageProvider` collects and flattens them into the final image which will be shown to user on the canvas(es).

The layer concept is under construction, so the documentation is unfinished yet.

### Accessing to its Processor

It is an easy task since it involves only calling the `GetProcessor` getter from anywhere inside the Layer's function. Calling this getter it's your responsibility to specify exactly that type of `Processor` which belongs to the current `Layer`. The return value of this function is a `std::shared_ptr` so handle it like a normal pointer type.

An example which calls the custom `CreateMagic()` function of your `FirstProcessor`:

```c++
GetProcessor<FirstProcessor>()->CreateMagic();
```

### Sequences

Each `Layer` has a number, called `Sequence`. This defines the order of layers to render on each other. You can query this number if you need by using member function `GetSequence` but please, by any means, do *not* ever change this value!

### Rendering

This is the function in you are free to leverage your creativity. One thing you need to pay a bit attention to: emit the *iAmReadyWithRendering* signal, here called `RenderReady` if you want your layer image to be shown. For example like this:

```c++
void FirstLayer::Render(const ProcessingStateDescriptor& psd)
{
    // Generate image to cv::Mat
    const cv::Mat image = psd.ExportToImage("");

    emit RenderReady(GetSequence(), image, GetOpacity(), true);
}
```

The signal signature is the following:
```c++
void RenderReady(const int layerSequence, // Sequence ID of the current layer
                 const cv::Mat& mat,      // The finished image in cv::Mat format
                 const float opacity,     // Opacity of the current layer set on the GUI
                 const bool dirty);       // true only if the cv::Mat has been changed
```

### Caching

Since converting many cv::Mats from the processors on every pixel the mouse advanced to can be a really time consuming operation, always use cache. If the `Layer` knows that the image didn't change, recall the previously rendered `cv::Mat` from member variable `Cache`. An example showing this:

```c++
void ShowImageLayer::Render(const ProcessingStateDescriptor &psd)
{
    bool dirty = true;

    if (Cache.data == nullptr) {
        cv::Mat imgOriginal = cv::imread(psd.ImageFileName, CV_LOAD_IMAGE_COLOR);
        Q_ASSERT(imgOriginal.data);

        Cache = imgOriginal;
    } else {
        dirty = false;
    }

    emit RenderReady(GetSequence(), Cache, dirty);
}
```

Beware that your condition on invalidating the cache might be different.

[QPoint]: http://doc.qt.io/qt-5/qpoint.html
[QMouseEvent]: http://doc.qt.io/qt-5/qmouseevent.html
[unordered_set]: http://www.cplusplus.com/reference/unordered_set/unordered_set/