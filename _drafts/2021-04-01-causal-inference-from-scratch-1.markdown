---
title: "Causal Inference from Scratch I: Getting Our Feet Wet"
date: 2021-04-01 12:00:00 -0700
comments: true
author: "Will High"
excerpt: Cau
toc: true
toc_sticky: true
classes: wide
---

Let's say $X$ causes $Y$.

<!-- 
https://tex.stackexchange.com/questions/45650/two-sets-of-aligned-equations-in-two-columns
-->
$$\begin{equation}
  \begin{split}
    X \to Y
  \end{split}
\phantom{\quad\leftrightarrow\quad}
  \begin{split}
    y & = f_X(x)
  \end{split}
\end{equation}$$

$$\require{AMScd}\begin{equation}\begin{CD}
cov(\mathcal{L}) @>>> non(\mathcal{K}) @>>> cf(\mathcal{K}) @>>>
cf(\mathcal{L})\\
@VVV @AAA @AAA @VVV\\
add(\mathcal{L}) @>>> add(\mathcal{K}) @>>> cov(\mathcal{K}) @>>>
non(\mathcal{L})
\end{CD}\end{equation}$$

$$\require{AMScd}\begin{equation}\begin{CD}
X @>>> Y 
\end{CD}\end{equation}$$

<script type="text/tikz">
  \begin{tikzpicture}
    \draw (0,0) circle (1in);
  \end{tikzpicture}
</script>


<script type="text/tikz">
  \begin{tikzpicture}
    \node[state] (1) {$Z$};
    \node[state] (2) [right =of 1] {$X$};
    \node[state] (3) [right =of 2] {$Y$};
    \node[state] (4) [above right =of 1,xshift=-0.3cm,yshift=-0.3cm] {$W$};

    \path (1) edge node[above] {$\lambda_{zx}$} (2);
    \path (2) edge node[above] {$\lambda_{xy}$} (3);
    \path[bidirected] (2) edge[bend left=60] node[above] {$\epsilon_{xy}$} (3);
    \path (4) edge node[el,above] {$\lambda_{wz}$} (1);
    \path[bidirected] (4) edge[bend left=50] node[el,above] {$\epsilon_{wy}$} (3);
  \end{tikzpicture}
</script>


# References

* [Causal Inference in Statistics: An Overview](https://ftp.cs.ucla.edu/pub/stat_ser/r350.pdf)
