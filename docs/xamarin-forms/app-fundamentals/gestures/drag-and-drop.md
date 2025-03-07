---
title: "Add drag and drop gesture recognizers"
description: "This article explains how to recognize drag and drop gestures with Xamarin.Forms."
ms.prod: xamarin
ms.assetid: 4CB2F270-908A-4A89-B852-70BC04066E8C
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 04/14/2021
no-loc: [Xamarin.Forms, Xamarin.Essentials]
---

# Add drag and drop gesture recognizers

[![Download Sample](~/media/shared/download.png) Download the sample](/samples/xamarin/xamarin-forms-samples/workingwithgestures-draganddropgesture/)

A drag and drop gesture enables items, and their associated data packages, to be dragged from one onscreen location to another location using a continuous gesture. Drag and drop can take place in a single application, or it can start in one application and end in another.

> [!IMPORTANT]
> Recognition of drag and drop gestures is supported on iOS, Android, and the Universal Windows Platform (UWP). However, on iOS a minimum platform of iOS 11 is required.

The *drag source*, which is the element on which the drag gesture is initiated, can provide data to be transferred by populating a data package object. When the drag source is released, drop occurs. The *drop target*, which is the element under the drag source, then processes the data package.

The process for enabling drag and drop in an application is as follows:

1. Enable drag on an element by adding a `DragGestureRecognizer` object to its `GestureRecognizers` collection. For more information, see [Enable drag](#enable-drag).
1. [optional] Build a data package. Xamarin.Forms automatically populates the data package for image and text controls, but for other content you'll need to construct your own data package. For more information, see [Build a data package](#build-a-data-package).
1. Enable drop on an element by adding a `DropGestureRecognizer` object its `GestureRecognizers` collection. For more information, see [Enable drop](#enable-drop).
1. [optional] Handle the `DropGestureRecognizer.DragOver` event to indicate the type of operation allowed by the drop target. For more information, see [Handle the DragOver event](#handle-the-dragover-event).
1. [optional] Process the data package to receive the dropped content. Xamarin.Forms will automatically retrieve image and text data from the data package, but for other content you'll need to process the data package. For more information, see [Process the data package](#process-the-data-package).

> [!NOTE]
> Dragging items to and from a [`CollectionView`](xref:Xamarin.Forms.CollectionView) is currently unsupported.

## Enable drag

In Xamarin.Forms, drag gesture recognition is provided by the `DragGestureRecognizer` class. This class defines the following properties:

- `CanDrag`, of type `bool`, which indicates whether the element the gesture recognizer is attached to can be a drag source. The default value of this property is `true`.
- `DragStartingCommand`, of type `ICommand`, which is executed when a drag gesture is first recognized.
- `DragStartingCommandParameter`, of type `object`, which is the parameter that's passed to the `DragStartingCommand`.
- `DropCompletedCommand`, of type `ICommand`, which is executed when the drag source is dropped.
- `DropCompletedCommandParameter`, of type `object`, which is the parameter that's passed to the `DropCompletedCommand`.

These properties are backed by [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) objects, which means that they can be targets of data bindings, and styled.

The `DragGestureRecognizer` class also defines `DragStarting` and `DropCompleted` events that fire provided that the `CanDrag` property is `true`. When a `DragGestureRecognizer` object detects a drag gesture, it executes the `DragStartingCommand` and invokes the `DragStarting` event. Then, when the `DragGestureRecognizer` object detects the completion of a drop gesture, it executes the `DropCompletedCommand` and invokes the `DropCompleted` event.

The `DragStartingEventArgs` object that accompanies the `DragStarting` event defines the following properties:

- `Handled`, of type `bool`, indicates whether the event handler has handled the event or whether Xamarin.Forms should continue its own processing.
- `Cancel`, of type `bool`, indicates whether the event should be canceled.
- `Data`, of type `DataPackage`, indicates the data package that accompanies the drag source. This is a read-only property.

The `DropCompletedEventArgs` object that accompanies the `DropCompleted` event has a read-only `DropResult` property, of type `DataPackageOperation`. For more information about the `DataPackageOperation` enumeration, see [Handle the DragOver event](#handle-the-dragover-event).

The following XAML example shows a `DragGestureRecognizer` attached to an [`Image`](xref:Xamarin.Forms.Image):

```xaml
<Image Source="monkeyface.png">
    <Image.GestureRecognizers>
        <DragGestureRecognizer />
    </Image.GestureRecognizers>
</Image>
```

In this example, a drag gesture can be initiated on the [`Image`](xref:Xamarin.Forms.Image).

> [!TIP]
> On iOS, Android, and UWP, a drag gesture is initiated with a long-press followed by a drag.

For an example of using `DragGestureRecognizer` commands, see the [sample](/samples/xamarin/xamarin-forms-samples/workingwithgestures-draganddropgesture/).

## Build a data package

Xamarin.Forms will automatically build a data package for you, when a drag is initiated, for the following controls:

- Text controls. Text values can be dragged from [`CheckBox`](xref:Xamarin.Forms.CheckBox), [`DatePicker`](xref:Xamarin.Forms.DatePicker), [`Editor`](xref:Xamarin.Forms.Editor), [`Entry`](xref:Xamarin.Forms.Entry), [`Label`](xref:Xamarin.Forms.Label), [`RadioButton`](xref:Xamarin.Forms.RadioButton), [`Switch`](xref:Xamarin.Forms.Switch), and [`TimePicker`](xref:Xamarin.Forms.TimePicker) objects.
- Image controls. Images can be dragged from [`Button`](xref:Xamarin.Forms.Button), [`Image`](xref:Xamarin.Forms.Image), and [`ImageButton`](xref:Xamarin.Forms.ImageButton) controls.

The following table shows the properties that are read, and any conversion that's attempted, when a drag is initiated on a text control:

| Control | Property | Conversion |
| --- | --- | --- |
| `CheckBox` | `IsChecked` | `bool` converted to a `string`. |
| `DatePicker` | `Date` | `DateTime` converted to a `string`. |
| `Editor` | `Text` ||
| `Entry` | `Text` ||
| `Label` | `Text` ||
| `RadioButton` | `IsChecked` | `bool` converted to a `string`. |
| `Switch` | `IsToggled` | `bool` converted to a `string`. |
| `TimePicker` | `Time` | `TimeSpan` converted to a `string`. |

For content other than text and images, you'll need to build a data package yourself.

Data packages are represented by the `DataPackage` class, which defines the following properties:

- `Properties`, of type `DataPackagePropertySet`, which is a collection of properties that comprise the data contained in the `DataPackage`. This property is a read-only property.
- `Image`, of type [`ImageSource`](xref:Xamarin.Forms.ImageSource), which is the image contained in the `DataPackage`.
- `Text`, of type `string`, which is the text contained in the `DataPackage`.
- `View`, of type `DataPackageView`, which is a read-only version of the `DataPackage`.

The `DataPackagePropertySet` class represents a property bag stored as a `Dictionary<string,object>`. For information about the `DataPackageView` class, see [Process the data package](#process-the-data-package).

### Store image or text data

Image or text data can be associated with a drag source by storing the data in the `DataPackage.Image` or `DataPackage.Text` property. This can be accomplished in the handler for the `DragStarting` event.

The following XAML example shows a `DragGestureRecognizer` that registers a handler for the `DragStarting` event:

```xaml
<Path Stroke="Black"
      StrokeThickness="4">
    <Path.GestureRecognizers>
        <DragGestureRecognizer DragStarting="OnDragStarting" />
    </Path.GestureRecognizers>
    <Path.Data>
        <!-- PathGeometry goes here -->
    </Path.Data>
</Path>
```

In this example, the `DragGestureRecognizer` is attached to a `Path` object. The `DragStarting` event is fired when a drag gesture is detected on the `Path`, which executes the `OnDragStarting` event handler:

```csharp
void OnDragStarting(object sender, DragStartingEventArgs e)
{
    e.Data.Text = "My text data goes here";
}
```

The `DragStartingEventArgs` object that accompanies the `DragStarting` event has a `Data` property, of type `DataPackage`. In this example, the `Text` property of the `DataPackage` object is set to a `string`. The `DataPackage` can then be accessed on drop, to retrieve the `string`.

### Store data in the property bag

Any data, including images and text, can be associated with a drag source by storing the data in the `DataPackage.Properties` collection. This can be accomplished in the handler for the `DragStarting` event.

The following XAML example shows a `DragGestureRecognizer` that registers a handler for the `DragStarting` event:

```xaml
<Rectangle Stroke="Red"
           Fill="DarkBlue"
           StrokeThickness="4"
           HeightRequest="200"
           WidthRequest="200">
    <Rectangle.GestureRecognizers>
        <DragGestureRecognizer DragStarting="OnDragStarting" />
    </Rectangle.GestureRecognizers>
</Rectangle>
```

In this example, the `DragGestureRecognizer` is attached to a `Rectangle` object. The `DragStarting` event is fired when a drag gesture is detected on the `Rectangle`, which executes the `OnDragStarting` event handler:

```csharp
void OnDragStarting(object sender, DragStartingEventArgs e)
{
    Shape shape = (sender as Element).Parent as Shape;
    e.Data.Properties.Add("Square", new Square(shape.Width, shape.Height));
}
```

The `DragStartingEventArgs` object that accompanies the `DragStarting` event has a `Data` property, of type `DataPackage`. The `Properties` collection of the `DataPackage` object, which is a `Dictionary<string, object>` collection, can be modified to store any required data. In this example, the `Properties` dictionary is modified to store a `Square` object, that represents the size of the `Rectangle`, against a "Square" key.

## Enable drop

In Xamarin.Forms, drop gesture recognition is provided by the `DropGestureRecognizer` class. This class defines the following properties:

- `AllowDrop`, of type `bool`, which indicates whether the element the gesture recognizer is attached to can be a drop target. The default value of this property is `true`.
- `DragOverCommand`, of type `ICommand`, which is executed when the drag source is dragged over the drop target.
- `DragOverCommandParameter`, of type `object`, which is the parameter that's passed to the `DragOverCommand`.
- `DragLeaveCommand`, of type `ICommand`, which is executed when the drag source is dragged off the drop target.
- `DragLeaveCommandParameter`, of type `object`, which is the parameter that's passed to the `DragLeaveCommand`.
- `DropCommand`, of type `ICommand`, which is executed when the drag source is dropped over the drop target.
- `DropCommandParameter`, of type `object`, which is the parameter that's passed to the `DropCommand`.

These properties are backed by [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) objects, which means that they can be targets of data bindings, and styled.

The `DropGestureRecognizer` class also defines `DragOver`, `DragLeave`, and `Drop` events that fire provided that the `AllowDrop` property is `true`. When a `DropGestureRecognizer` recognizes a drag source over the drop target, it executes the `DragOverCommand` and invokes the `DragOver` event. Then, if the drag source is dragged off the drop target, the `DropGestureRecognizer` executes the `DragLeaveCommand` and invokes the `DragLeave` event. Finally, when the `DropGestureRecognizer` recognizes a drop gesture over the drop target, it executes the `DropCommand` and invokes the `Drop` event.

The `DragEventArgs` class, that accompanies the `DragOver` and `DragLeave` events, defines the following properties:

- `Data`, of type `DataPackage`, which contains the data associated with the drag source. This property is read-only.
- `AcceptedOperation`, of type `DataPackageOperation`, which specifies which operations are allowed by the drop target.

For information about the `DataPackageOperation` enumeration, see [Handle the DragOver event](#handle-the-dragover-event).

The `DropEventArgs` class that accompanies the `Drop` event defines the following properties:

- `Data`, of type `DataPackageView`, which is a read-only version of the data package.
- `Handled`, of type `bool`, indicates whether the event handler has handled the event or whether Xamarin.Forms should continue its own processing.

The following XAML example shows a `DropGestureRecognizer` attached to an [`Image`](xref:Xamarin.Forms.Image):

```xaml
<Image BackgroundColor="Silver"
       HeightRequest="300"
       WidthRequest="250">
    <Image.GestureRecognizers>
        <DropGestureRecognizer />
    </Image.GestureRecognizers>
</Image>
```

In this example, when a drag source is dropped on the [`Image`](xref:Xamarin.Forms.Image) drop target, the drag source will be copied to the drop target, provided that the drag source is an [`ImageSource`](xref:Xamarin.Forms.ImageSource). This occurs because Xamarin.Forms automatically copies dragged images, and text, to compatible drop targets.

For an example of using `DropGestureRecognizer` commands, see the [sample](/samples/xamarin/xamarin-forms-samples/workingwithgestures-draganddropgesture/).

## Handle the DragOver event

The `DropGestureRecognizer.DragOver` event can be optionally handled to indicate which type of operations are allowed by the drop target. This can be accomplished by setting the `AcceptedOperation` property, of type `DataPackageOperation`, of the `DragEventArgs` object that accompanies the `DragOver` event.

The `DataPackageOperation` enumeration defines the following members:

- `None`, indicates that no action will be performed.
- `Copy`, indicates that the drag source content will be copied to the drop target.

> [!IMPORTANT]
> When a `DragEventArgs` object is created, the `AcceptedOperation` property defaults to `DataPackageOperation.Copy`.

The following XAML example shows a `DropGestureRecognizer` that registers a handler for the `DragOver` event:

```xaml
<Image BackgroundColor="Silver"
       HeightRequest="300"
       WidthRequest="250">
    <Image.GestureRecognizers>
        <DropGestureRecognizer DragOver="OnDragOver" />
    </Image.GestureRecognizers>
</Image>
```

In this example, the `DropGestureRecognizer` is attached to an [`Image`](xref:Xamarin.Forms.Image) object. The `DragOver` event is fired when a drag source is dragged over the drop target, but hasn't been dropped, which executes the `OnDragOver` event handler:

```csharp
void OnDragOver(object sender, DragEventArgs e)
{
    e.AcceptedOperation = DataPackageOperation.None;
}
```

In this example, the `AcceptedOperation` property of the `DragEventArgs` object is set to `DataPackageOperation.None`. This ensures that no action is taken when a drag source is dropped over the drop target.

## Process the data package

The `Drop` event is fired when a drag source is released over a drop target. When this occurs, Xamarin.Forms will automatically attempt to retrieve data from the data package, when a drag source is dropped onto the following controls:

- Text controls. Text values can be dropped onto [`CheckBox`](xref:Xamarin.Forms.CheckBox), [`DatePicker`](xref:Xamarin.Forms.DatePicker), [`Editor`](xref:Xamarin.Forms.Editor), [`Entry`](xref:Xamarin.Forms.Entry), [`Label`](xref:Xamarin.Forms.Label), [`RadioButton`](xref:Xamarin.Forms.RadioButton), [`Switch`](xref:Xamarin.Forms.Switch), and [`TimePicker`](xref:Xamarin.Forms.TimePicker) objects.
- Image controls. Images can be dropped onto [`Button`](xref:Xamarin.Forms.Button), [`Image`](xref:Xamarin.Forms.Image), and [`ImageButton`](xref:Xamarin.Forms.ImageButton) controls.

The following table shows the properties that are set, and any conversion that's attempted, when a text-based drag source is dropped on a text control:

| Control | Property | Conversion |
| --- | --- | --- |
| `CheckBox` | `IsChecked` | `string` is converted to a `bool`. |
| `DatePicker` | `Date` | `string` is converted to a `DateTime`. |
| `Editor` | `Text` ||
| `Entry` | `Text` ||
| `Label` | `Text` ||
| `RadioButton` | `IsChecked` | `string` is converted to a `bool`. |
| `Switch` | `IsToggled` | `string` is converted to a `bool`. |
| `TimePicker` | `Time` | `string` is converted to a `TimeSpan`. |

For content other than text and images, you'll need to process the data package yourself.

The `DropEventArgs` class that accompanies the `Drop` event defines a `Data` property, of type `DataPackageView`. This property represents a read-only version of the data package.

### Retrieve image or text data

Image or text data can be retrieved from a data package in the handler for the `Drop` event, using methods defined in the `DataPackageView` class.

The `DataPackageView` class includes `GetImageAsync` and `GetTextAsync` methods. The `GetImageAsync` method retrieves an image from the data package, that was stored in the `DataPackage.Image` property, and returns `Task<ImageSource>`. Similarly, the `GetTextAsync` method retrieves text from the data package, that was stored in the `DataPackage.Text` property, and returns `Task<string>`.

The following example shows a `Drop` event handler that retrieves text from the data package for a `Path`:

```csharp
async void OnDrop(object sender, DropEventArgs e)
{
    string text = await e.Data.GetTextAsync();

    // Perform logic to take action based on the text value.
}
```

In this example, text data is retrieved from the data package using the `GetTextAsync` method. An action based on the text value can then be taken.

### Retrieve data from the property bag

Any data can be retrieved from a data package in the handler for the `Drop` event, by accessing the `Properties` collection of the data package.

The `DataPackageView` class defines a `Properties` property, of type `DataPackagePropertySetView`. The `DataPackagePropertySetView` class represents a read-only property bag stored as a `Dictionary<string, object>`.

The following example shows a `Drop` event handler that retrieves data from the property bag of a data package for a `Rectangle`:

```csharp
void OnDrop(object sender, DropEventArgs e)
{
    Square square = (Square)e.Data.Properties["Square"];

    // Perform logic to take action based on retrieved value.
}
```

In this example, the `Square` object is retrieved from the property bag of the data package, by specifying the "Square" dictionary key. An action based on the retrieved value can then be taken.

## Related links

- [Drag and drop gesture (sample)](/samples/xamarin/xamarin-forms-samples/workingwithgestures-draganddropgesture/)
