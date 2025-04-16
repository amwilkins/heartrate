<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" lang="en" xml:lang="en"><head>

<meta charset="utf-8">
<meta name="generator" content="quarto-1.7.23">

<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">


</head>

<body class="fullcontent quarto-light">

<div id="quarto-content" class="page-columns page-rows-contents page-layout-article">

<main class="content" id="quarto-document-content">

<header id="title-block-header" class="quarto-title-block default">
<div class="quarto-title">
<h1 class="title">FitBit Heart Rate Data</h1>
</div>



<div class="quarto-title-meta">

    
  
    
  </div>
  


</header>


<p><sub> Updated 2025-04-16 </sub></p>
<p>Heartrate data is gathered roughly every 15 seconds by the FitBit Inspire 2, model FB418. It’s unknown whether this is a snapshot, or window average.</p>
<section id="data" class="level2">
<h2 class="anchored" data-anchor-id="data">Data</h2>
<div id="b14d6fe1" class="cell" data-execution_count="2">
<details class="code-fold">
<summary>Code</summary>
<div class="sourceCode cell-code" id="cb1"><pre class="sourceCode python code-with-copy"><code class="sourceCode python"><span id="cb1-1"><a href="#cb1-1" aria-hidden="true" tabindex="-1"></a><span class="co"># Don't use scientific notation</span></span>
<span id="cb1-2"><a href="#cb1-2" aria-hidden="true" tabindex="-1"></a>pd.options.display.float_format <span class="op">=</span> <span class="st">'</span><span class="sc">{:.0f}</span><span class="st">'</span>.<span class="bu">format</span></span>
<span id="cb1-3"><a href="#cb1-3" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb1-4"><a href="#cb1-4" aria-hidden="true" tabindex="-1"></a><span class="cf">if</span> os.path.exists(<span class="st">"./data/heart_rate_combined.pkl"</span>):</span>
<span id="cb1-5"><a href="#cb1-5" aria-hidden="true" tabindex="-1"></a>    df <span class="op">=</span> pd.read_pickle(<span class="st">"./data/heart_rate_combined.pkl"</span>)</span>
<span id="cb1-6"><a href="#cb1-6" aria-hidden="true" tabindex="-1"></a><span class="cf">else</span>:</span>
<span id="cb1-7"><a href="#cb1-7" aria-hidden="true" tabindex="-1"></a>    file_list <span class="op">=</span> glob.glob(<span class="st">"./fitbit_data/heart_rate-*.json"</span>)</span>
<span id="cb1-8"><a href="#cb1-8" aria-hidden="true" tabindex="-1"></a>    data_frames <span class="op">=</span> [pd.read_json(<span class="bu">file</span>) <span class="cf">for</span> <span class="bu">file</span> <span class="kw">in</span> file_list]</span>
<span id="cb1-9"><a href="#cb1-9" aria-hidden="true" tabindex="-1"></a>    df <span class="op">=</span> pd.concat(data_frames)</span>
<span id="cb1-10"><a href="#cb1-10" aria-hidden="true" tabindex="-1"></a>    df[<span class="st">'dateTime'</span>] <span class="op">=</span> df[<span class="st">'dateTime'</span>].dt.tz_localize(<span class="st">"UTC"</span>)</span>
<span id="cb1-11"><a href="#cb1-11" aria-hidden="true" tabindex="-1"></a>    df[<span class="st">'dateTime'</span>] <span class="op">=</span> df[<span class="st">'dateTime'</span>].dt.tz_convert(<span class="st">"US/Central"</span>)</span>
<span id="cb1-12"><a href="#cb1-12" aria-hidden="true" tabindex="-1"></a>    df[<span class="st">'year-month'</span>] <span class="op">=</span> df[<span class="st">'dateTime'</span>].dt.strftime(<span class="st">"%Y-%m"</span>)</span>
<span id="cb1-13"><a href="#cb1-13" aria-hidden="true" tabindex="-1"></a>    df[<span class="st">'year'</span>] <span class="op">=</span> df[<span class="st">'dateTime'</span>].dt.year</span>
<span id="cb1-14"><a href="#cb1-14" aria-hidden="true" tabindex="-1"></a>    df[<span class="st">'month'</span>] <span class="op">=</span> df[<span class="st">'dateTime'</span>].dt.month</span>
<span id="cb1-15"><a href="#cb1-15" aria-hidden="true" tabindex="-1"></a>    df[<span class="st">'day'</span>] <span class="op">=</span> df[<span class="st">'dateTime'</span>].dt.day</span>
<span id="cb1-16"><a href="#cb1-16" aria-hidden="true" tabindex="-1"></a>    df[<span class="st">"bpm"</span>] <span class="op">=</span> [i[<span class="st">"bpm"</span>] <span class="cf">for</span> i <span class="kw">in</span> df[<span class="st">"value"</span>]]</span>
<span id="cb1-17"><a href="#cb1-17" aria-hidden="true" tabindex="-1"></a>    df <span class="op">=</span> df.drop(columns<span class="op">=</span>[<span class="st">"value"</span>])</span>
<span id="cb1-18"><a href="#cb1-18" aria-hidden="true" tabindex="-1"></a>    df.to_pickle(<span class="st">"./data/heart_rate_combined.pkl"</span>)</span></code><button title="Copy to Clipboard" class="code-copy-button"><i class="bi"></i></button></pre></div>
</details>
</div>
<p>Data is held as a series of daily json files, which are loaded and concatenated into a Pandas dataframe. The dataframe consists of the datetime in ISO format and bpm, with this analysis consisting of 9962774 datapoints. Other fields are created for ease of parsing later.</p>
<p>A sample of the data format is provided here:</p>
<div id="29468a53" class="cell" data-execution_count="3">
<div class="cell-output cell-output-stdout">
<pre><code>                       dateTime year-month  year  month  day  bpm
10680 2024-06-06 23:59:38-05:00    2024-06  2024      6    6   83
10681 2024-06-06 23:59:43-05:00    2024-06  2024      6    6   82
10682 2024-06-06 23:59:58-05:00    2024-06  2024      6    6   83</code></pre>
</div>
</div>
<!-- Possible dates to mark:   -->
<!-- 2022-12-02 - 2022-12-11 - Spain Vacation   -->
<!-- 2024-05-05 - 2024-05-13 - Italy Vacation   -->
<!-- 2024-06-01 - Ongoing - Move into new apart``ment   -->
<!-- 2024-12-07 - 2024-12-14 - Canada Vacation   -->
</section>
<section id="plotting" class="level2">
<h2 class="anchored" data-anchor-id="plotting">Plotting:</h2>
<div id="d11153f9" class="cell" data-execution_count="4">
<details class="code-fold">
<summary>Code</summary>
<div class="sourceCode cell-code" id="cb3"><pre class="sourceCode python code-with-copy"><code class="sourceCode python"><span id="cb3-1"><a href="#cb3-1" aria-hidden="true" tabindex="-1"></a><span class="im">from</span> plotnine <span class="im">import</span> <span class="op">*</span></span>
<span id="cb3-2"><a href="#cb3-2" aria-hidden="true" tabindex="-1"></a><span class="im">from</span> mizani.breaks <span class="im">import</span> date_breaks</span>
<span id="cb3-3"><a href="#cb3-3" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb3-4"><a href="#cb3-4" aria-hidden="true" tabindex="-1"></a><span class="co"># Prep daily df</span></span>
<span id="cb3-5"><a href="#cb3-5" aria-hidden="true" tabindex="-1"></a>daily <span class="op">=</span> df.groupby([<span class="st">'year'</span>,<span class="st">'month'</span>,<span class="st">'day'</span>])[<span class="st">'bpm'</span>].mean().reset_index()</span>
<span id="cb3-6"><a href="#cb3-6" aria-hidden="true" tabindex="-1"></a>daily[<span class="st">'date'</span>] <span class="op">=</span> daily[<span class="st">'year'</span>].astype(<span class="bu">str</span>) <span class="op">+</span> <span class="st">"-"</span> <span class="op">+</span>  daily[<span class="st">'month'</span>].astype(<span class="bu">str</span>) <span class="op">+</span>  <span class="st">"-"</span> <span class="op">+</span> daily[<span class="st">'day'</span>].astype(<span class="bu">str</span>)</span>
<span id="cb3-7"><a href="#cb3-7" aria-hidden="true" tabindex="-1"></a>daily[<span class="st">'date'</span>] <span class="op">=</span> pd.to_datetime(daily[<span class="st">'date'</span>])</span>
<span id="cb3-8"><a href="#cb3-8" aria-hidden="true" tabindex="-1"></a>daily <span class="op">=</span> daily[[<span class="st">'date'</span>,<span class="st">'bpm'</span>]]</span>
<span id="cb3-9"><a href="#cb3-9" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb3-10"><a href="#cb3-10" aria-hidden="true" tabindex="-1"></a><span class="co"># Plot</span></span>
<span id="cb3-11"><a href="#cb3-11" aria-hidden="true" tabindex="-1"></a>(</span>
<span id="cb3-12"><a href="#cb3-12" aria-hidden="true" tabindex="-1"></a>    ggplot(daily, aes(x<span class="op">=</span><span class="st">"date"</span>, y<span class="op">=</span><span class="st">"bpm"</span>, group <span class="op">=</span> <span class="dv">1</span>))</span>
<span id="cb3-13"><a href="#cb3-13" aria-hidden="true" tabindex="-1"></a>    <span class="op">+</span> geom_line()</span>
<span id="cb3-14"><a href="#cb3-14" aria-hidden="true" tabindex="-1"></a>    <span class="op">+</span> scale_x_date(breaks<span class="op">=</span>date_breaks(width<span class="op">=</span><span class="st">"3 months"</span>),date_minor_breaks<span class="op">=</span><span class="st">"3 months"</span>)</span>
<span id="cb3-15"><a href="#cb3-15" aria-hidden="true" tabindex="-1"></a>    <span class="op">+</span> geom_smooth(span<span class="op">=</span><span class="fl">.2</span>)</span>
<span id="cb3-16"><a href="#cb3-16" aria-hidden="true" tabindex="-1"></a>    <span class="op">+</span> labs(x <span class="op">=</span> <span class="st">"Date"</span>, </span>
<span id="cb3-17"><a href="#cb3-17" aria-hidden="true" tabindex="-1"></a>      y <span class="op">=</span> <span class="st">"Heartrate in BPM"</span>, </span>
<span id="cb3-18"><a href="#cb3-18" aria-hidden="true" tabindex="-1"></a>      title <span class="op">=</span> <span class="st">"Daily Average Heartrate"</span>)</span>
<span id="cb3-19"><a href="#cb3-19" aria-hidden="true" tabindex="-1"></a>        <span class="op">+</span> theme(</span>
<span id="cb3-20"><a href="#cb3-20" aria-hidden="true" tabindex="-1"></a>            axis_text_x<span class="op">=</span>element_text(rotation<span class="op">=</span><span class="dv">25</span>, hjust<span class="op">=</span><span class="dv">1</span>),</span>
<span id="cb3-21"><a href="#cb3-21" aria-hidden="true" tabindex="-1"></a>            panel_grid<span class="op">=</span>element_line(color<span class="op">=</span><span class="st">"grey"</span>)</span>
<span id="cb3-22"><a href="#cb3-22" aria-hidden="true" tabindex="-1"></a>         )    </span>
<span id="cb3-23"><a href="#cb3-23" aria-hidden="true" tabindex="-1"></a>        <span class="op">+</span> geom_vline(xintercept<span class="op">=</span>[dt.date(<span class="dv">2024</span>,<span class="dv">5</span>,<span class="dv">1</span>)], linetype<span class="op">=</span><span class="st">"dashed"</span>, color<span class="op">=</span><span class="st">"grey"</span>)</span>
<span id="cb3-24"><a href="#cb3-24" aria-hidden="true" tabindex="-1"></a>        <span class="op">+</span> annotate(geom<span class="op">=</span><span class="st">"label"</span>,fill<span class="op">=</span><span class="st">"grey"</span>,color<span class="op">=</span><span class="st">"black"</span>,label<span class="op">=</span><span class="st">"Moved to St. Louis"</span>, x<span class="op">=</span>dt.date(<span class="dv">2024</span>,<span class="dv">5</span>,<span class="dv">1</span>), y<span class="op">=</span><span class="dv">108</span>)</span>
<span id="cb3-25"><a href="#cb3-25" aria-hidden="true" tabindex="-1"></a>)</span></code><button title="Copy to Clipboard" class="code-copy-button"><i class="bi"></i></button></pre></div>
</details>
<div class="cell-output cell-output-display">
<div>
<figure class="figure">
<p><img src="Heartrate_files/figure-html/cell-5-output-1.png" width="672" height="480" class="figure-img"></p>
</figure>
</div>
</div>
</div>
<div id="b85cf4fc" class="cell" data-execution_count="5">
<details class="code-fold">
<summary>Code</summary>
<div class="sourceCode cell-code" id="cb4"><pre class="sourceCode python code-with-copy"><code class="sourceCode python"><span id="cb4-1"><a href="#cb4-1" aria-hidden="true" tabindex="-1"></a><span class="im">import</span> pandas <span class="im">as</span> pd</span>
<span id="cb4-2"><a href="#cb4-2" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb4-3"><a href="#cb4-3" aria-hidden="true" tabindex="-1"></a>df[<span class="st">'hour'</span>] <span class="op">=</span> df[<span class="st">'dateTime'</span>].dt.hour</span>
<span id="cb4-4"><a href="#cb4-4" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb4-5"><a href="#cb4-5" aria-hidden="true" tabindex="-1"></a>hourly <span class="op">=</span> df.loc[df[<span class="st">'year'</span>] <span class="op">==</span> <span class="dv">2024</span>].groupby([<span class="st">'hour'</span>], as_index<span class="op">=</span> <span class="va">False</span>)[<span class="st">'bpm'</span>].mean()</span>
<span id="cb4-6"><a href="#cb4-6" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb4-7"><a href="#cb4-7" aria-hidden="true" tabindex="-1"></a>(</span>
<span id="cb4-8"><a href="#cb4-8" aria-hidden="true" tabindex="-1"></a>    ggplot(hourly[[<span class="st">'hour'</span>,<span class="st">'bpm'</span>]], aes(x<span class="op">=</span><span class="st">"hour"</span>, y<span class="op">=</span><span class="st">"bpm"</span>, group <span class="op">=</span> <span class="dv">1</span>))</span>
<span id="cb4-9"><a href="#cb4-9" aria-hidden="true" tabindex="-1"></a>    <span class="op">+</span> geom_line()</span>
<span id="cb4-10"><a href="#cb4-10" aria-hidden="true" tabindex="-1"></a>        <span class="op">+</span> scale_x_continuous(limits<span class="op">=</span>(<span class="dv">0</span>,<span class="dv">24</span>),breaks<span class="op">=</span><span class="bu">range</span>(<span class="dv">0</span>, <span class="dv">24</span>, <span class="dv">1</span>))</span>
<span id="cb4-11"><a href="#cb4-11" aria-hidden="true" tabindex="-1"></a>    <span class="op">+</span> labs(x <span class="op">=</span> <span class="st">"Hour"</span>, </span>
<span id="cb4-12"><a href="#cb4-12" aria-hidden="true" tabindex="-1"></a>      y <span class="op">=</span> <span class="st">"Heartrate in BPM"</span>, </span>
<span id="cb4-13"><a href="#cb4-13" aria-hidden="true" tabindex="-1"></a>      title <span class="op">=</span> <span class="st">"Hourly Average Heartrate over 2024"</span>)</span>
<span id="cb4-14"><a href="#cb4-14" aria-hidden="true" tabindex="-1"></a>        <span class="op">+</span> theme_538()</span>
<span id="cb4-15"><a href="#cb4-15" aria-hidden="true" tabindex="-1"></a>)</span></code><button title="Copy to Clipboard" class="code-copy-button"><i class="bi"></i></button></pre></div>
</details>
<div class="cell-output cell-output-display">
<div>
<figure class="figure">
<p><img src="Heartrate_files/figure-html/cell-6-output-1.png" width="672" height="480" class="figure-img"></p>
</figure>
</div>
</div>
</div>
</section>
<section id="last-month-data" class="level2">
<h2 class="anchored" data-anchor-id="last-month-data">Last month data</h2>
<div id="fba879b2" class="cell" data-execution_count="6">
<details class="code-fold">
<summary>Code</summary>
<div class="sourceCode cell-code" id="cb5"><pre class="sourceCode python code-with-copy"><code class="sourceCode python"><span id="cb5-1"><a href="#cb5-1" aria-hidden="true" tabindex="-1"></a>last_month <span class="op">=</span> df.loc[df[<span class="st">'year-month'</span>] <span class="op">==</span> <span class="st">"2024-03"</span>].groupby([<span class="st">'hour'</span>], as_index<span class="op">=</span> <span class="va">False</span>)[<span class="st">'bpm'</span>].mean()</span>
<span id="cb5-2"><a href="#cb5-2" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb5-3"><a href="#cb5-3" aria-hidden="true" tabindex="-1"></a>(</span>
<span id="cb5-4"><a href="#cb5-4" aria-hidden="true" tabindex="-1"></a>    ggplot(last_month[[<span class="st">'hour'</span>,<span class="st">'bpm'</span>]], aes(x<span class="op">=</span><span class="st">"hour"</span>, y<span class="op">=</span><span class="st">"bpm"</span>, group <span class="op">=</span> <span class="dv">1</span>))</span>
<span id="cb5-5"><a href="#cb5-5" aria-hidden="true" tabindex="-1"></a>    <span class="op">+</span> geom_line()</span>
<span id="cb5-6"><a href="#cb5-6" aria-hidden="true" tabindex="-1"></a>        <span class="op">+</span> scale_x_continuous(limits<span class="op">=</span>(<span class="dv">0</span>,<span class="dv">24</span>),breaks<span class="op">=</span><span class="bu">range</span>(<span class="dv">0</span>, <span class="dv">24</span>, <span class="dv">1</span>))</span>
<span id="cb5-7"><a href="#cb5-7" aria-hidden="true" tabindex="-1"></a>    <span class="op">+</span> labs(x <span class="op">=</span> <span class="st">"Hour"</span>, </span>
<span id="cb5-8"><a href="#cb5-8" aria-hidden="true" tabindex="-1"></a>      y <span class="op">=</span> <span class="st">"Heartrate in BPM"</span>, </span>
<span id="cb5-9"><a href="#cb5-9" aria-hidden="true" tabindex="-1"></a>      title <span class="op">=</span> <span class="st">"Hourly Average Heartrate March 2024"</span>)</span>
<span id="cb5-10"><a href="#cb5-10" aria-hidden="true" tabindex="-1"></a>        <span class="op">+</span> theme_538()</span>
<span id="cb5-11"><a href="#cb5-11" aria-hidden="true" tabindex="-1"></a>)</span>
<span id="cb5-12"><a href="#cb5-12" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb5-13"><a href="#cb5-13" aria-hidden="true" tabindex="-1"></a></span>
<span id="cb5-14"><a href="#cb5-14" aria-hidden="true" tabindex="-1"></a><span class="co"># (</span></span>
<span id="cb5-15"><a href="#cb5-15" aria-hidden="true" tabindex="-1"></a><span class="co">#     ggplot(hourly[['hour','bpm']], aes(x="hour", y="bpm", group = 1))</span></span>
<span id="cb5-16"><a href="#cb5-16" aria-hidden="true" tabindex="-1"></a><span class="co">#     + geom_line()</span></span>
<span id="cb5-17"><a href="#cb5-17" aria-hidden="true" tabindex="-1"></a><span class="co">#       + scale_x_continuous(limits=(0,24),breaks=range(0, 24, 1))</span></span>
<span id="cb5-18"><a href="#cb5-18" aria-hidden="true" tabindex="-1"></a><span class="co">#     + labs(x = "Hour", </span></span>
<span id="cb5-19"><a href="#cb5-19" aria-hidden="true" tabindex="-1"></a><span class="co">#       y = "Heartrate in BPM", </span></span>
<span id="cb5-20"><a href="#cb5-20" aria-hidden="true" tabindex="-1"></a><span class="co">#       title = "Hourly Average Heartrate over 2024")</span></span>
<span id="cb5-21"><a href="#cb5-21" aria-hidden="true" tabindex="-1"></a><span class="co">#       + theme_538()</span></span>
<span id="cb5-22"><a href="#cb5-22" aria-hidden="true" tabindex="-1"></a><span class="co"># )</span></span></code><button title="Copy to Clipboard" class="code-copy-button"><i class="bi"></i></button></pre></div>
</details>
<div class="cell-output cell-output-display">
<div>
<figure class="figure">
<p><img src="Heartrate_files/figure-html/cell-7-output-1.png" width="672" height="480" class="figure-img"></p>
</figure>
</div>
</div>
</div>
</section>

</main>
