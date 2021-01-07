# Datenstruktur und Algorithmen

## Karatsuba and Ofman algorithm


$$
\begin{align*}
(10^{n/2} a + b) \cdot (10 ^{n/2} c + d)
&= 10^n \textbf{a $\cdot$ c }+ 10 ^{n/2} \textbf{a $\cdot$ c} + 10 ^{n/2}\textbf{ b $\cdot$ d} + \textbf{b $\cdot$ d }+ 10 ^{n/2} \textbf{(a − b) $\cdot$ (d − c)}\\
&=(10^n + 10 ^{n/2}) \cdot a  c + (10 ^{n/2}+ 1 ) \cdot{b  d }+ 10 ^{n/2}\cdot {(a − b)  (d − c)}\\
\end{align*}
$$
