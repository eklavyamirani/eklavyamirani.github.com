---
layout: post
title: Easy Xamarin.Forms Container To Support Pinch To Zoom in Content in iOS using Custom Renderers
category: Xamarin.Forms
tags: [Xamarin.Forms, Xamarin, iOS, Custom Renderer]
meta: {}
author: Eklavya Mirani
---
{% include JB/setup %}
Very recently I started working with Xamarin.Forms and I am enjoying hacking my way through the intricacies of the framework.
I have been creating a lot of custom controls for things such as Horizontal ListView, Tabbed Views for iOS, and other complex controls. So while developing one of these controls, I wanted them to have support for Pinch To Zoom gesture. Currently, as of this post, Xamarin.Forms has support for only one gesture - "Tap". This left me with two choices: Re-Invent the wheel and write the Gesture Recognizer on my own (or find somebody's implementation of the same) or use the native Gesture Recognizer.

I chose the second one. From my experience with Windows Store App development, I know that the ScrollView inherenetly supports pinch to zoom capability. I checked out iOS' UIScrollView for the same and it seems like if I wrap my view around a UIScrollView in iOS/ScrollView in WinRT, I should be able to achieve the desired functionality without having to write a lot of code.

So I wrote an empty container to Wrap around my view that is rendered using Native controls that support pinch to zoom capability out of the box.
This is what my implementation looks like:
{% highlight csharp %}
public class PinchToZoomContainer : View
{
    public View Content { get; set; }
}
{% endhighlight %}

As simple as it gets. We could have directly used a ContentView here, without the need for a Content property in view. *Makes Sense*

But the thing is, I will render the view myself, so I don't want the content to be rendered by Xamarin. Otherwise, I end up with two views, one rendered by xamarin, the other by myself. Looks crappy, trust me.

Coming to the iOS implementation:

{% highlight csharp%}
[assembly:ExportRenderer(typeof(PinchToZoomContainer), typeof(PinchToZoomRenderer))]
namespace TestPinchToZoom.iOS
{
    public class PinchToZoomRenderer : ViewRenderer<PinchToZoomContainer, UIScrollView>
    {
        private View Content;
        private bool childrenCreated = false;

        protected override void OnElementChanged (ElementChangedEventArgs<PinchToZoomContainer> e)
        {
            base.OnElementChanged (e);
            
            if (e.NewElement == null) {
                return;
            }
            
            if (Control == null) {
                SetNativeControl (new UIScrollView ());
            }
            
            // Allow the Control to support pinch to zoom
            Control.MaximumZoomScale = 3f;
            Control.MinimumZoomScale = 0.1f;
            Control.ViewForZoomingInScrollView += GetViewForZoomingInScrollView;
            this.Content = e.NewElement.Content;
        }

        private UIView GetViewForZoomingInScrollView(UIScrollView scrollView)
        {
            return scrollView.Subviews.FirstOrDefault ();
        }
        
        public override void LayoutSubviews ()
        {
            base.LayoutSubviews ();
            if (Control == null || Control.Subviews == null) 
            {
                return;
            }
        
            if (!childrenCreated) 
            {
                // render the child Content of the view
                var childRenderer = RendererFactory.GetRenderer (this.Content);
                // This is important. The renderer requires the size of the element to render.
                childRenderer.SetElementSize (new Size (Control.Frame.Width, Control.Frame.Height));
                var nativeChildView = childRenderer.NativeView;
                // Again, we need to provide a frame, since we are laying out the view
                nativeChildView.Frame = Control.Frame;
                Control.AddSubview (nativeChildView);
                this.childrenCreated = true;
            }
        
        }
    }
}
{% endhighlight %}

This code is pretty much self explanatory. The whole idea is to render the content while laying out the subview. At this point our parent view is already rendered and we just create a new native view from the Xamarin.Forms view and add it as a "Subview", i.e. child to the parent pinch to zoom container.