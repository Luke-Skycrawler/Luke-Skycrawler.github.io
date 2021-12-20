---
title: Auto-Quantization in Physical Simulations
date: 2021-12-17 16:32:17
tags: taichi
thumbnail: 
# - /img/auto_quant/4quant.jpg
- /img/auto_quant/fluid.jpg

---

In this paper, we propose a novel workflow to allow users to seamlessly tune their simulations for a smaller memory footprint, which is especially valuable for scaling the simulation to a higher resolution. Given an objective for compression rate or relative error bound, the whole process is automated as part of the compiler support, and each identified variable is statically assigned to an appropriate length-optimized type. Referencing the derivatives supplied by auto-diff support, we proposed a precise estimation of the round-off errors with respect to quantization schemes, allowing us to achieve much higher efficiency than Random Search in determining the quantization schemes. We implemented several examples to demonstrate the generality and efficacy of our method, in which the average compression rate is 1.9, without degradation of perceived physical feasibility in visual effects.  


(Coming soon)
<!-- more -->
{% raw %}
<video src="/videos/compression.mp4" type='video/mp4' controls='controls' width='100%' height='100%'>
</video>
{% endraw%} 
**Top left:** float64. **Top right:** compression rate 1.7. **Bottom left:** compression rate 2.0. **Bottom right:** compression rate 2.5. 

<!-- 
### 1. Introduction
Previously, QuanTaichi \cite{hu2021quantaichi} implemented a compiler-supported "quantized" type system, providing user control over representations of physical variables in memory to achieve a higher information density. 

Undoubtedly, the quantized type is a desirable invention for large-scale simulations with high resolutions. For the simulations on a single GPU, the introduction of lower-precision("quantized") types breaks the hard limit on the simulated particle count posed by the amount of available device-side memory by as much as 1.7 times. On the other hand, multiple-GPU simulations are benefited both on scale and speed, as the quantized types save a similar proportion on the bandwidth consumption. 

Reportedly, researchers are beginning to tap into the potential of quantizing in the field of artificial intelligence, where exhilarating results manifest that a quantized 8-bit neural network is x-times faster than its floating-point counterpart in training, yet still achieving the same level of accuracy \cite{gholami2021survey}.

However, we are not safe to assume the quantization as a 'harmless' gift until we find the answer to a series of questions on the influence of quantization on the simulated process. How much further can we compress the variables before the quality loss become intolerable, and how many bits of information are essential to encode the whole process? 
With the current "quantaichi" implementation, users have to tackle these core issues themselves, most likely by trial-and-error and completely void of systematic assistance.

Except for the efforts of automatic decision of fixed-point scheme \cite{wang2011automated} in computer-aided hardware design, few existing attempts have been made on automatic customization of bit length in the field of computer simulations. 
The problem is weaved with the following troublesome characteristic: 

(1) Quantization error tends to accumulate over time, eventually hijacking the ensuing simulation. Nevertheless, there are a series of simulations that are sensitive to the initial condition. As a result, point-to-point correspondence in the final result is almost impossible, giving rise to the necessity of an alternative definition of the loss of visual effect.

(2) As the number of the customized types grows, the search space fluctuates exponentially. This curse of dimension makes fine-grained customization almost impossible.

(3) The sparsity of the controlled variable in the decision of fraction bits. It means that we must either solve a discrete optimization or apply relaxation to the original problem.

Inspired by the Error Propagation formula, we consult the derivatives to estimate the impact of quantization error. We generalized the workflow to a compiler-supported automated quantizing support.
With our system, users can effortlessly achieve a human-compatible compression rate with a physically plausible visual effect. 

To achieve those benefits, users must specify a key measurement $z$ to indicate simulation quality, e.g. the energy function for energetic liquid simulation. Depending on their emphasis, users can choose their focus between fidelity and compression rate, by either appointing a relative error constrain or setting a target compression rate. 

Due to the precise estimation of quantization error, our method has the following benefits to distinguish itself from existing quantizing frameworks:

\textbf{Quantitative:} Depending on users' emphasis, we provide quantitative guarantees on the quality of the resultant quantizing scheme. If fidelity is the focus, our typical solution achieves a distribution of an average and standard variance of xxx and xxx times of error bound respectively. Or, for the user-specified compression rate, we guarantee to meet the constrain with minor exceptions due to up-rounding of the required fraction bits.

\textbf{Time-efficient:} We are able to arrive at a fair combination after a single run of the simulation, whereas the final solution lies only one or two bits away. For the specified error bound to hold, the median of iteration times for simulation is xxx.



Collectively, our main contributions are:

(1) An general error analysis scheme to precisely estimate quantization error relying on the gradients.

(2) Introduction of dithering, an efficient technique for controlling quantization error in scalable simulations.

(3) A simple compiler interface for users to automatically decide and apply quantization to the variables and an enhancement on the current automatic differentiation implementation to lower the threshold of our prerequisites. 

(4) A series of experiments to substantiate the scalability of our quantization scheme and to provide suggestions for constraint target functions.


Our system is implemented as an extension to the Taichi programming language\cite{hu2019taichi} to take advantage of its autodiff feature. However, our methods are general for simulations implemented on any differentiable programming systems. -->

