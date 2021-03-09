Time as a Signal
================

<script>
    init_alia_demos(['time-signal', 'simple-animation',
        'number-smoothing', 'color-smoothing',
        'flickering-demo', 'deflickering-demo']);
</script>

Although all signals in alia are conceptually time-varying values, most of them
only care about the present (e.g., `a + b` is just whatever `a` is right now
plus whatever `b` is). However, sometimes we want our signals to explicitly
vary with time (for example, for animation purposes). This section discusses
some of the utilities that alia provides for that purpose.

The Time Signal
---------------

The most basic way to work with time is to access alia's time signal. This is
just a simple millisecond counter:

```cpp
html::p(ctx,
    alia::printf(ctx,
        "It's been %d seconds since you started looking at this page.",
        get_animation_tick_count(ctx) / 1000));
```

<div class="demo-panel">
<div id="time-signal"></div>
</div>

Calling `get_animation_tick_count` will cause alia to request that the
underlying system issue a new refresh event (after a small animation-friendly
delay) and will also ensure that any component calling it gets refreshed as
part of that event. This means that you can very easily create animations using
the animation tick count:

```cpp
colored_box(
    ctx,
    interpolate(
        rgb8(255, 255, 255),
        rgb8(0, 118, 255),
        (1 + std::sin(get_raw_animation_tick_count(ctx) / 600.)) / 2));
```

<div class="demo-panel">
<div id="simple-animation"></div>
</div>

Note that we use `get_raw_animation_tick_count` here. This returns the tick
count as a raw integer value. The benefits of working with signals are less
applicable when working with the animation tick count because a) the tick count
is always available and b) it's always changing, so there's little point in
trying to cache results and detect changes.

Of course, since accessing the animation tick count causes your UI to
continuously refresh, you should be careful when using it to develop web pages
or desktop apps that don't normally do this. Some of the other time-related
utilities in alia are provided specifically because they minimize unnecessary
refreshes.

Smoothing
---------

A common case where you want to make explicit use of time is to provide a
smoothed view of a signal as it changes. alia provides the `smooth` function
for this purpose:

<dl>

<dt>smooth(ctx, x, transition = default_transition)</dt><dd>

`smooth(ctx, x, transition)`, where `x` is a signal, yields another signal that
carries a smoothed view of the value in `x`. When `x` changes abruptly, the
smoothed view will change smoothly according to `transition`.

</dd>

</dl>

See `alia/timing/smoothing.hpp` for details on specifying custom transitions,
abruptly changing a smoothed signal, and the interfaces required of a value
that needs to be smoothed.

Here's an example of basic number smoothing:

```cpp
html::p(ctx, "Enter N:");
html::input(ctx, n);
html::button(ctx, "1", n <<= 1);
html::button(ctx, "100", n <<= 100);
html::button(ctx, "10000", n <<= 10000);
html::p(ctx, "Here's a smoothed view of N:");
html::p(ctx, smooth(ctx, n));
```

<div class="demo-panel">
<div id="number-smoothing"></div>
</div>

One option for making your value compatible with smoothing is to simply provide
the basic arithmetic operators. Here's a smoothed view of a color (with a
custom transition thrown in as well):

```cpp
colored_box(
    ctx, smooth(ctx, color, animated_transition{ease_in_out_curve, 200}));
html::button(ctx, "Go Light", color <<= rgb8(210, 210, 220));
html::button(ctx, "Go Dark", color <<= rgb8(50, 50, 55));
```

<div class="demo-panel">
<div id="color-smoothing"></div>
</div>

Deflickering
------------

Sometimes your signals will suffer from flickering. This is common on signals
that are calculated (or retrieved) in the background, especially when the
signal is updated based on some input from the user. For signals like this, the
default behavior is to appear empty while the value is being calculated, and
when the calculation is nearly instantaneous, the user will perceive this as
flickering.

alia provides a utility for deflickering a signal:

<dl>

<dt>deflicker(ctx, x, delay = default_deflicker_delay)</dt><dd>

`deflicker(ctx, x, delay)`, where `x` is a signal, yields another signal that
carries a deflickered view of the value in `x`. If `x` loses its value for less
than `delay` milliseconds, the deflickered view will *maintain the last value
that `x` carried.* (If `x` doesn't take on a new value before the delay
elapses, then the deflickered view will also lose its value.)

</dd>

</dl>

To see `deflicker()` in action, let's first create a signal that flickers. We
can do this artificially using `alia::async` and a simple delayed callback...

```cpp
html::p(ctx, "Enter N:");
html::input(ctx, n);
html::button(ctx, "++", ++n);
html::p(ctx, "Here's a flickering signal carrying N * 2:");
html::p(ctx,
    alia::async<int>(ctx,
        [](auto ctx, auto reporter, auto n) {
            html::async_call([=]() {
                reporter.report_success(n * 2);
            }, 200);
        },
        n));
```

<div class="demo-panel">
<div id="flickering-demo"></div>
</div>

Now let's add a call to `deflicker`:

```cpp
html::p(ctx, "Enter N:");
html::input(ctx, n);
html::button(ctx, "++", ++n);
html::p(ctx, "Here's the same signal from above with deflickering:");
html::p(ctx,
    deflicker(ctx,
        alia::async<int>(ctx,
            [](auto ctx, auto reporter, auto n) {
                html::async_call([=]() {
                    reporter.report_success(n * 2);
                }, 200);
            },
            n)));
```

<div class="demo-panel">
<div id="deflickering-demo"></div>
</div>
