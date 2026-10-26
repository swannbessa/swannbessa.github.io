---
layout: default
title: "Dino10B README"
permalink: /blog/dino10b/readme/
---

<style>
  .dino10b-tabs { display: flex; gap: 0; border-bottom: 1px solid var(--rule); margin: 0 0 22px; }
  .dino10b-tabs a { padding: 6px 14px; margin-bottom: -1px; color: var(--muted, #6b7280);
                    border-bottom: 2px solid transparent; font-family: var(--mono); font-size: 0.9em; }
  .dino10b-tabs a.active { color: var(--text); border-bottom-color: var(--text); font-weight: 600; }
  .dino10b a[href^="#ref"] { color: inherit; border-bottom: none; }
  .dino10b a.ext { color: #0b5fa5; border-bottom: none; text-decoration: underline; }
  .dino10b img { max-width: 100%; display: block; margin: 1.4rem auto 0.3rem; }
  .dino10b p > em:only-child { display: block; text-align: center; margin: 0 0 1.4rem; }
  .dino10b pre { background: #f2f2f0; border: 1px solid var(--rule); border-radius: 6px;
                 padding: 12px 16px; overflow-x: auto; font-size: 0.85em; scroll-margin-top: 2rem; }
  .dino10b pre code { background: none; border: none; padding: 0; }
  .dino10b .source-note { color: var(--muted, #6b7280); font-size: 0.9em; }
</style>

{% raw %}
<div class="dino10b">
<nav class="dino10b-tabs"><a href="/blog/dino10b/">Blog post</a><a href="/blog/dino10b/readme/" class="active">README</a></nav>
<p class="source-note">The dataset's own README, also available at <a class="ext" href="https://dl.fbaipublicfiles.com/large_objects/dino_vitl_10B/README.md">https://dl.fbaipublicfiles.com/large_objects/dino_vitl_10B/README.md</a>.</p>
<h1>DatasetDINO10B</h1>
<p>A simple interface for loading a dataset of DINO patch descriptors for similarity search benchmarks.</p>
<p>WARNING : When using large batch sizes, prefer batch sizes that are divisors of 200,000,000, which is the number of vectors per .bvecs file. This will avoid splitting batches between two .bvecs files, causing significant overhead due to concate&gt;</p>
<h2>Downloading the data</h2>
<p>The fastest way (by far) to download the large amount of data (10 TB) is to use the Axel Download accelerator (download from https://github.com/axel-download-accelerator/axel). Put the following code in a .sh file and execute it in your <code>data</code> folder to get the full dataset (see Dataset class in Faiss about the <code>data</code> folder). Beware, the downloading might take several days due to the large size of the data files (10TB). 
If you want only a subset of the dataset, you can selectively download a part of the dataset. The dataset is divided into 50 chunks of name <code>chunk_i.bvecs</code> which are listed in the list of files <code>file_list.txt</code> (downloaded in the first wget command in the code below).
Each chunk file has 200 GB of data, and contains 200M vectors. If you just want, for example, a dataset of 400M vectors, you can edit the <code>file_list.txt</code> file to only download the first two chunks <code>chunk_0000.bvecs</code> and <code>chunk_0001.bvecs</code>.</p>
<pre id="download-script"><code class="language-bash">#!/bin/bash
axel https://dl.fbaipublicfiles.com/large_objects/dino_vitl_10B/file_list.txt

base_dir=&quot;dino_vitl_10B&quot;
mkdir -p &quot;$base_dir&quot;

while read -r url; do
  # strip scheme+host
  path=${url#*://*/}
  path=$(echo &quot;$url&quot; | sed -E 's|https?://[^/]+/||')
  # remove first two dirs (large_objects/dino_vitl_10B)
  rel=$(echo &quot;$path&quot; | cut -d/ -f3-)
  [ -z &quot;$rel&quot; ] &amp;&amp; rel=$(basename &quot;$path&quot;)
  out=&quot;$base_dir/$rel&quot;
  mkdir -p &quot;$(dirname &quot;$out&quot;)&quot;
  axel -n 16 -a -o &quot;$out&quot; &quot;$url&quot;
done &lt; file_list.txt
rm file_list.txt
</code></pre>
<p>Alternatively you can use a simple wget script but it might take several days to complete :</p>
<pre id="download-script-wget"><code class="language-bash">#!/bin/bash
wget https://dl.fbaipublicfiles.com/large_objects/dino_vitl_10B/file_list.txt
wget -i file_list.txt --force-directories --cut-dirs=2
mv dl.fbaipublicfiles.com dino_vitl_10B
rm file_list.txt
</code></pre>
<h2>Usage</h2>
<p>If you use the code from faiss library :</p>
<pre><code class="language-python">from faiss.contrib.datasets import DatasetDINO10B

# Initialize with dataset size. The data must be in a `data` folder in your working directory.
ds = DatasetDINO10B(
    nb=1_000_000,  # number of base vectors
)
</code></pre>
<p>If you use the <code>code_dataset_dino.py</code> file downloaded from fbaipublicfiles, import the class using :</p>
<pre><code class="language-python">from code_dataset_dino import DatasetDINO10B
ds = DatasetDINO10B(
    nb = 1_000_000,
    path = &quot;path/to/the/downloaded/folder/dino_vitl_10B&quot;
)
</code></pre>
<h2>Supported Dataset Sizes</h2>
<ul>
<li>100,000</li>
<li>200,000</li>
<li>500,000</li>
<li>1,000,000</li>
<li>2,000,000</li>
<li>5,000,000</li>
<li>10,000,000</li>
<li>20,000,000</li>
<li>50,000,000</li>
<li>100,000,000</li>
<li>200,000,000</li>
<li>500,000,000</li>
<li>1,000,000,000</li>
<li>2,000,000,000</li>
<li>5,000,000,000</li>
<li>10,000,000,000</li>
</ul>
<h2>File Structure</h2>
<p>The dataset path should contain:
- directory <code>chunked_base_10B</code> - base vectors
- <code>queries_clean.bvecs</code> - test queries<br />
- <code>train_queries_99M.bvecs</code> - training queries
- <code>gts/gts_dino_patch_{nb}_k10.npy</code> - ground truth nearest neighbors</p>
</div>
{% endraw %}
