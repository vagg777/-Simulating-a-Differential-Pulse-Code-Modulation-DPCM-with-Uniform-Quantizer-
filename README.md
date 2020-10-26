# Simulating a  Differential Pulse-Code Modulation (DPCM) with Uniform Quantizer 

## 1. Introduction

**DPCM (Differential Pulse Code Modulation)** can be considered as a generalization of Delta coding where the signal quantized and sent to the receiver is the difference between the current sample (time n) and a linear prediction. That is, in DPCM coding, we calculate, at each time point, a prediction for the value of the current sample based on the values of previous samples that have already been coded, and then calculate the error of that prediction. The prediction error signal is then encoded using one or more bits per sample.

## 2. Tools needed

* Project Version : `Final`
* IDE Version : `Matlab`
* Programming Language : `Matlab`
* Matlab Version : `2019b`

## 3. How DPCM works
The encoder and decoder of a DPCM system are shown in ***Figure 1***. In order to encode the value of the current sample we first calculate a forecast for its value based on coded values of previous samples. The prediction of the signal `x(n)` 𝑥is denoted as `y'(n)`. In ***Figure 1***, we observe a memory device (both in the transmitter and in the receiver), which keeps stored the reconstructed values of the previous samples based on which the prediction of the price of the current sample will be calculated. Our aim is to minimize the scatter of the error signal `y(n) = x(n) - y'(n)`, so that it has a small dynamic range and can be satisfactorily described by a small number of bits. The process of quantizing the error signal `y' = x(n)` leads to the signal `y^(n)` which is sent to the receiver.

![This is a alt text.](https://i.ibb.co/Mgt3bxN/1.png "Figure 1")

***Figure 1***

In the receiver, the signal `y^(n)` is combined with the signal `y'(n)` (the prediction of `x(n)`). Since the previously reconstructed values, as well as the forecasting method used by the transmitter, are known to the receiver, it implies that the transmitter and receiver are able to calculate exactly the same values for the prediction `y'(n)`. As in the case of Δελτα (Delta) encoding, here too the transmitter includes as part of the receiver device which calculates the reconstruction `x^(n)`. These values are used by the transmitter to calculate the prediction, and not the actual values `x(n)`, in order to fully mimic the layout of the receiver which of course does not know the actual values. By using the reconstructed values to calculate the prediction and then the prediction error, we ensure (as in the case of delta coding) that we do not have accumulation of the quantization error.

In the simple case, where we rely only on the prediction of the previous sample, the equations which describe the operation of the DPCM system of ***Figure 1*** are the following:

1.  `y(n) = x(n) - y'(n)` 
2.  `y^(n) = Q(y(n))`
3. `y^'(n) = y^(n) + y'(n)`

where
* `y'(n) = y^'(n-1)` in the case of using a previous sample
* `Q(·)`is the input-output function of the gradient quantifier (uniform) used. 

From the above relations, we obtain the expression for the quantization error:

4.  `𝑦𝑄(𝑛) = x^(n) - x(n) = y^(n) - y(n)`

We observe that if in the above equations we set `y^'(n) = 0`, that is a DPCM system which does not use prediction, then this system is equivalent to a simple PCM coding system.

## 4. Building the Uniform Bit Quantizer
The uniform bit quantizer, as the name suggests, quantizes the prediction error which has a smaller dynamic range than the input signal. The quantizer in particular will quantize each sample of the prediction error separately and will be implemented as a `MATLAB` function, as follows:

`y^(n) = my_quantizer( y(n), N, min_value, max_value)` 

, where:
*  `y^(n)` is the current sample of the prediction error as quantum input
* `N` is the number of bits to be used
* `min_value` is the minimum acceptable value of the forecast error (proposed value is `min_value = -3.5`)
* `max_value` is the maximum acceptable value of the forecast error (proposed value is `max_value = +3.5`)
*  `y^(n)` is the quantized sample of the current sample of the forecast error

The quantization levels are represented by the integers `1,2,…., 2𝑁`  where the largest positive quantization level corresponds to the integer 1. These integers can be represented binarily with binary digits.

Moreover, `centers` is a vector with the centers of the quantization areas. In particular, the quantist should limit the dynamic range of the prediction error to the values `[min_value : max_value]` by placing the samples outside the dynamic range at the corresponding extreme acceptable value. The quantizer then calculates the quantization step `Δ`, the centers of each region, calculates to which region the input sample belongs, and outputs the coded sample `y^(n)`. The coded sample at the output of the quantizer will take values between `1,2,…., 2𝑁`, which are the quantization regions. The quantized version of the output sample will take the value of the quantization center of the area to which the current input sample belongs. This sample will be used as a pointer to the vector `centers` get the quantized sample as `centers(y^(n))`. An example of the quantizer areas for `N = 2` are depicted below in ***Figure 2***. 

![This is a alt text.](https://i.ibb.co/grH1p2D/Screenshot-1.png "Figure 2")

***Figure 2***

## 5. Loading the source
The source to be encoded/decoded is a `10,000` samples signal. The samples of the source we were experimenting shows good predictability, i.e. a current sample of it can be predicted (in the statistical sense) with a small prediction error combining previous values of the same signal. Samples of the source to be experimented with are stored in the file named `source.mat`. To retrieve the input data just type:

` > load source.mat`

## 6. Running the DPCM system
1. We build the aforementioned system in `MATLAB`.
2. We choose two values of `p ≥ 4` and for `N = 1,2,3 bits`, we draw the initial signal and the prediction error on the same graph
    - Indicative examples for `p = 4` and `N = 1,2,3 bits` (blue color: input signal, orange color: error prediction signal)
        ![This is a alt text.](https://i.ibb.co/kcqZsbB/P-4-N-1.png "Figure 3")

        ***Figure 3. p=4 and N=1***
        
        ![This is a alt text.](https://i.ibb.co/ftNXP3C/P-4-N-2.png "Figure 4")
        
        ***Figure 4. p=4 and N=2***
   
        ![This is a alt text.](https://i.ibb.co/60fH8F3/P-4-N-3.png "Figure 5")
   
        ***Figure 5. p=4 and N=3***
    - As `N` increases, the reconstructed y-chart has fewer and fewer differences-anomalies with the initial x, i.e. it tends to get closer and closer to the initial x. 
    - Also, the various values of `p` for constant `N` have little effect on the reconstructed y-chart. 
    - Therefore, the higher the values of `p` and `N`, the better the initial signal is represented at the output of the decoder (ie at the output of the system). 
    - This is because the higher the `p`, the more information will be stored in memory, so more efficient prediction is made and passed to the reconstructed signal. 
    - Also, the larger the `N`, the smaller the prediction error, so the smaller the error in the reconstructed signal, as the quantizer will have more quantization areas and will be better represented with a smaller quantization error.
3. We evaluate its performance with a graph showing the mean square prediction error with respect to `N` and for various values of `p`. Specifically for number of bits `N = 1,2,3 bits` used by the uniform quantizer to encode the prediction signal and for a predicted order `p = 4: 1: 8`.
    - Indicative result:
    
        ![This is a alt text.](https://i.ibb.co/DtJhmt2/Q3.png "Figure 6")


        ***Figure 6. Mean square prediction error for all p values***
    - Drawing the graph `E(Y^2) - N` for the various values of `p` with respect to `N`, we observe that indeed the square prediction error `E(Y^2)` decreases slightly when we change `p` and without necessarily following any ascending or descending order for `E` and we observe multiple similar alternations in maximum values or the same N as the value of p changes. 
    - Again, the essential changes in the value of `E` come from the change of values in `N` and not in `p`, which is also shown in the diagram below. 
    - In any case, the higher the `p`, the smaller the error, although from one point onwards, increasing the `p` does not help to substantially minimize the error `E`.

4. For `N = 1,2,3 bits`, we display the original and the reconstructed signal on the receiver for `p = 4 and p = 8` and we discuss upon on the reconstruction results in relation to the quantization bits.
    - Indicative examples for `p = 4` and `N = 1,2,3 bits` (blue color: input signal, orange color: decoded output signal)
        ![This is a alt text.](https://i.ibb.co/gm03DcT/p-4-N-1.png "Figure 7")

        ***Figure 7. p=4 and N=1***
        
        ![This is a alt text.](https://i.ibb.co/JcBmsdV/P-4-N-2.png "Figure 8")
        
        ***Figure 8. p=4 and N=2***
   
        ![This is a alt text.](https://i.ibb.co/kX0c384/p-4-N-3.png "Figure 9")
   
        ***Figure 9. p=4 and N=3***
    - Once again, we see the difference created by the different values of `N`. 
    - As `N` increases, the reconstructed y-chart has fewer and fewer differences-anomalies with the initial x, i.e. it tends to get closer and closer to the initial signal. 
    - Also, the various values of `p` for constant `N` have little effect on the reconstructed y-chart. 
    - Therefore, the higher the values of `p` and `N`, the better the initial signal is represented at the output of the decoder.
    - This is because the higher the `p`, the more information will be stored in memory, so more efficient prediction is made and passed to the reconstructed signal. 
    - Also, the higher the `N`, the smaller the prediction error, so the smaller the error in the reconstructed signal, as the quantizer will have more quantization areas and will be better represented with a smaller quantization error.
