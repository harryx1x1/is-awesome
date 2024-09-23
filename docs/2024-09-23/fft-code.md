# FFT Code

## Recursion version
```rust
use num::complex::Complex;

use std::f64::consts::PI;

  

// Base function for both FFT and IFFT

fn fft_base(coeffs: Vec<Complex<f64>>, is_ifft: bool) -> Vec<Complex<f64>> {

let n = coeffs.len();

if n == 1 {

return coeffs;

}

  

// Split the input into even and odd indices

let even_coeffs: Vec<Complex<f64>> = coeffs.iter().step_by(2).cloned().collect();

let odd_coeffs: Vec<Complex<f64>> = coeffs.iter().skip(1).step_by(2).cloned().collect();

  

// Calculate the twiddle factor (omega)

// For IFFT, use the conjugate of the normal FFT twiddle factor

let theta = if is_ifft { -2.0 * PI / n as f64 } else { 2.0 * PI / n as f64 };

let omega = Complex::from_polar(1.0, theta);

  

// Recursive calls for even and odd parts

let y_even = fft_base(even_coeffs, is_ifft);

let y_odd = fft_base(odd_coeffs, is_ifft);

  

// Combine the results using the butterfly operation

let mut y = vec![Complex::new(0.0, 0.0); n];

let mut current_omega = Complex::new(1.0, 0.0);

for k in 0..n/2 {

let t = current_omega * y_odd[k];

y[k] = y_even[k] + t;

y[k + n/2] = y_even[k] - t;

current_omega *= omega;

}

y

}

  

// Fast Fourier Transform (FFT) function

fn fft(coeffs: Vec<Complex<f64>>) -> Vec<Complex<f64>> {

fft_base(coeffs, false)

}

  

// Inverse Fast Fourier Transform (IFFT) function

fn ifft(coeffs: Vec<Complex<f64>>) -> Vec<Complex<f64>> {

let n = coeffs.len() as f64;

// Perform IFFT and normalize the result by dividing by n

fft_base(coeffs, true).iter().map(|x| x / n).collect()

}

  

#[cfg(test)]

mod tests {

use super::*;

  

#[test]

fn test_fft_ifft() {

// create input data

let input: Vec<Complex<f64>> = vec![1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0, 13.0, 14.0, 15.0, 16.0]

.into_iter()

.map(|x| Complex::new(x, 0.0))

.collect();

  

// execute FFT

let fft_result = fft(input.clone());

println!("fft_result: {:?}", fft_result);

// execute IFFT

let ifft_result = ifft(fft_result);

println!("ifft_result: {:?}", ifft_result);

// verify IFFT result is close to original input

for (original, result) in input.iter().zip(ifft_result.iter()) {

assert!((original - result).norm() < 1e-10, "IFFT result does not match original input");

}

}

}
```

## Iterative version

```rust
use num::complex::Complex;

use std::f64::consts::PI;

  

// Fast Fourier Transform (FFT) function

fn fft(mut input: Vec<Complex<f64>>) -> Vec<Complex<f64>> {

let n = input.len();

if n <= 1 {

return input;

}

  

// Bit-reversal permutation

for i in 0..n {

let j = reverse_bits(i, n);

if i < j {

input.swap(i, j);

}

}

  

// Cooley-Tukey FFT algorithm (butterfly operations)

let mut step = 1;

while step < n {

let jump = step * 2;

let omega = Complex::from_polar(1.0, -PI / step as f64);

let mut w = Complex::new(1.0, 0.0);

  

for group in 0..step {

for pair in (group..n).step_by(jump) {

let t = input[pair + step] * w;

input[pair + step] = input[pair] - t;

input[pair] = input[pair] + t;

}

w *= omega;

}

  

step *= 2;

}

  

input

}

  

// Inverse Fast Fourier Transform (IFFT) function

fn ifft(mut input: Vec<Complex<f64>>) -> Vec<Complex<f64>> {

let n = input.len();

if n <= 1 {

return input;

}

  

// Bit-reversal permutation

for i in 0..n {

let j = reverse_bits(i, n);

if i < j {

input.swap(i, j);

}

}

  

// Cooley-Tukey IFFT algorithm (butterfly operations)

let mut step = 1;

while step < n {

let jump = step * 2;

let omega = Complex::from_polar(1.0, PI / step as f64); // Note the sign change compared to FFT

let mut w = Complex::new(1.0, 0.0);

  

for group in 0..step {

for pair in (group..n).step_by(jump) {

let t = input[pair + step] * w;

input[pair + step] = input[pair] - t;

input[pair] = input[pair] + t;

}

w *= omega;

}

  

step *= 2;

}

  

// Normalization

let scale = 1.0 / n as f64;

input.iter_mut().for_each(|x| *x *= scale);

  

input

}

  

// Bit reversal function

fn reverse_bits(mut num: usize, bit_count: usize) -> usize {

let mut reversed = 0;

for _ in 0..bit_count.trailing_zeros() {

reversed = (reversed << 1) | (num & 1);

num >>= 1;

}

reversed

}

  

#[cfg(test)]

mod tests {

use super::*;

  

#[test]

fn test_fft_ifft() {

// create input data

let input: Vec<Complex<f64>> = vec![1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0, 11.0, 12.0, 13.0, 14.0, 15.0, 16.0]

.into_iter()

.map(|x| Complex::new(x, 0.0))

.collect();

  

// execute FFT

let fft_result = fft(input.clone());

println!("fft_result: {:?}", fft_result);

// execute IFFT

let ifft_result = ifft(fft_result);

println!("ifft_result: {:?}", ifft_result);

// verify IFFT result is close to original input

for (original, result) in input.iter().zip(ifft_result.iter()) {

assert!((original - result).norm() < 1e-10, "IFFT result does not match original input");

}

}

}
```