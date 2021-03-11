Assorted Examples
=================

<script>
    init_alia_demos(['for-each-map-demo', 'time-signal',
        'number-smoothing', 'color-smoothing', 'factor-tree']);
</script>

If you're more interested in code than prose, you'll like this page the best.
Many of these come from other sections of the documentation.

You may also want to check out the [alia/HTML demos](https://html.alia.dev/).

Containers
----------

Here's an (admittedly contrived) example of working with containers in alia.
It uses a `std::map` to map player names to their scores.

Like the tip calculator on the landing page, it also shows off some of the more
basic features of alia, like [actions](actions.md) and how you can use alia's
data graph to 'magically' manifest state when and where you need it, even in
the middle of declarative component code.

```cpp
void
scoreboard(html::context ctx, duplex<std::map<std::string, int>> scores)
{
    alia::for_each(ctx, scores,
        [&](auto player, auto score) {
            html::scoped_div div(ctx, "item");
            html::element(ctx, "h4").text(player);
            html::p(ctx, alia::printf(ctx, "%d points", score));
            html::button(ctx, "GOAL!", ++score);
        });

    auto new_player = alia::get_state(ctx, std::string());
    html::input(ctx, new_player);
    html::button(ctx, "Add Player",
        (scores[new_player] <<= alia::mask(0, new_player != ""),
         new_player <<= ""));
}
```

<div class="demo-panel">
<div id="for-each-map-demo"></div>
</div>

Factor Trees
------------

This example displays factorization trees for numbers that you enter. (It
implements a poor man's tree view by allowing the user to show and hide the
subtree associated with each composite factor.)

What's interesting about this is that there's actually no application data that
mirrors this tree view. The application 'model' consists entirely of a single
integer. The structure of the UI (and the state that allows the user to expand
and collapse nodes in the tree) is fully defined by the recursive structure of
the calls to `factor_tree`.

```cpp
void
factor_tree(html::context ctx, readable<int> n)
{
    html::scoped_div div(ctx, "subtree");

    // Get the 'best' factor that n has. (The one closest to sqrt(n).)
    auto f = alia::apply(ctx, factor, n);

    // If that factor is 1, n is prime.
    alia_if(f != 1)
    {
        html::p(ctx, alia::printf(ctx, "%i: composite", n));

        // Allow the user to expand this block to see more factor.
        auto expanded = get_state(ctx, false);
        html::button(ctx,
            conditional(expanded, "Hide Factors", "Show Factors"),
            actions::toggle(expanded));
        alia_if(expanded)
        {
            factor_tree(ctx, f);
            factor_tree(ctx, n / f);
        }
        alia_end
    }
    alia_else
    {
        html::p(ctx, alia::printf(ctx, "%i: prime", n));
    }
    alia_end
}

// And here's the demo UI that invokes the top-level of the tree:
void
factor_tree_demo(html::context ctx, duplex<int> n)
{
    html::p(ctx, "Enter a number:");
    html::input(ctx, n);
    factor_tree(ctx, n);
}
```

<div class="demo-panel">
<div id="factor-tree"></div>
</div>

Timing Signals
--------------

Although all signals in alia are conceptually time-varying values, most of them
only care about the present (e.g., `a + b` is just whatever `a` is right now
plus whatever `b` is). However, some signals are more closely linked to time
and explicitly vary with it:

```cpp
html::p(ctx,
    alia::printf(ctx,
        "It's been %d seconds since you started looking at this page.",
        get_animation_tick_count(ctx) / 1000));
```

<div class="demo-panel">
<div id="time-signal"></div>
</div>

You can use this explicit notion of time to do fun things like smooth out other
signals:

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

You can smooth anything that provides the basic arithmetic operators. Here's a
smoothed view of a color with a custom transition:

```cpp
colored_box(
    ctx, smooth(ctx, color, animated_transition{ease_in_out_curve, 200}));
html::button(ctx, "Go Light", color <<= rgb8(210, 210, 220));
html::button(ctx, "Go Dark", color <<= rgb8(50, 50, 55));
```

<div class="demo-panel">
<div id="color-smoothing"></div>
</div>
