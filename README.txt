===========================================================
 Temporary Market Impact Modeling from Order Book Data
===========================================================

This script provides a Python implementation for modeling the temporary market
impact of a financial instrument directly from a snapshot of its limit order book (LOB).

It serves two primary functions:
1. It calculates the exact, "book-implied" slippage curve by simulating an
   aggressive order that sweeps the available liquidity.
2. It fits this empirical curve to a robust, depth-scaled power-law model,
   producing a few simple parameters that summarize the book's liquidity profile.

This model is a critical component for estimating execution costs (slippage) and
serves as a foundational block for building optimal execution algorithms.


-----------------------
Core Concepts
-----------------------

1. Book-Implied Impact Curve (g_t(x))
   This is the "ground-truth" instantaneous impact. It is calculated by finding the
   Volume-Weighted Average Price (VWAP) for executing an order of size `x` and
   comparing it to the mid-price. The result is a discrete, piecewise-constant
   function representing the real cost structure of the current order book.

2. Depth-Scaled Power-Law Model
   To create a smooth, continuous, and regularized model, we fit the book-implied
   curve to the following functional form:

   g(x) ≈ s/2 + θ * (x / D)^φ

   Where:
   - g(x):       The estimated slippage (cost per share) for an order of size `x`.
   - s/2:        The half-spread, representing the minimum cost for a tiny order.
   - D:          The local depth scale (e.g., total size in the top 3 levels).
   - θ (theta):  The fitted impact coefficient (a scaling factor).
   - φ (phi):    The fitted impact exponent, capturing the curvature (e.g., φ=1 is
                 linear impact, φ=0.5 is square-root).


-----------------------
How to Run
-----------------------

1. Prerequisites:
   - Python 3.x
   - NumPy library

   If you do not have NumPy, install it via pip:
   pip install numpy

2. Execution:
   Run the script directly from your terminal:
   python your_script_name.py


-----------------------
Code Structure
-----------------------

- OrderBookSnapshot: A simple dataclass to hold the ask/bid prices and sizes.
- _vwap_from_book(): A core helper function that calculates the VWAP for consuming
  a given quantity from one side of the book.
- slippage_curve_from_book(): Generates the empirical slippage curve over a grid
  of order sizes.
- fit_depth_scaled_power_law(): Fits the power-law model to the empirical curve
  using log-linear least-squares regression.
- Main Block (`if __name__ == "__main__"`): Contains a toy example demonstrating
  how to use the functions and prints the final fitted parameters.


-----------------------
Example Output
-----------------------

Running the script with the default toy data will produce the following output,
showing the key book metrics and the fitted model parameters for both sides:

Mid-Price=80.9500, Half-Spread=0.0200, D_buy=373, D_sell=450

Buy-Side Fit (g ≈ H + θ(x/D)^φ):
  θ=0.0345, φ=1.5033, RMSE=0.000028

Sell-Side Fit (g ≈ H + θ(x/D)^φ):
  θ=0.0121, φ=1.7088, RMSE=0.000009


-----------------------
How to Use With Your Own Data
-----------------------

1. Locate the `if __name__ == "__main__"` block at the bottom of the script.
2. Replace the hard-coded `ask_prices`, `ask_sizes`, `bid_prices`, and `bid_sizes`
   lists with your own order book data. This data can be loaded from a CSV file,
   a database, or a live data feed.
3. Adjust the `top_k` parameter in the `ob.depth_scale()` calls if you wish to
   define market depth differently.
4. Run the script to get the fitted impact parameters for your instrument.
