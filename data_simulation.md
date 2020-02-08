Simulation
================
Qi Yuchen
2020/2/7

# Generate data

n\_sample = 1000 \# fix sample size

n\_parameter and prop\_strong may be changed.

Default of correlation is set to be 0.3, maybe it should be other number
like 0.5.

Default of coef\_strong is 2.

X follows N(0,1).

``` r
set.seed(12345)

sim_data = function(n_sample = 200, n_parameter = 50, prop_strong = 0.1, prop_wbc = 0.2, prop_wai = 0.2, c = 1, cor = 0.30, coef_strong = 2) {
  # Numbers of four signals
  n_strong = as.integer(n_parameter * prop_strong) # strong
  n_wbc = as.integer(n_parameter * prop_wbc) # weak but correlated
  n_wai = as.integer(n_parameter * prop_wai) # weak and independent
  n_null = n_parameter - n_strong - n_wbc - n_wai # null
  
  if (n_null < 0) {
    return("Given parameters' proportions are not valid.")
  }
  
  bound = c * sqrt(log(n_parameter) / n_sample) # threshold of weak/strong, the default is 0.14
  if (coef_strong < bound) {
    coef_strong = coef_strong + 2 * bound
  }
  
  cor_matrix = diag(n_parameter)
  # wbc covariates are correlated to the first strong covariate
  cor_matrix[1, (n_strong + n_wai + 1):(n_strong + n_wai + n_wbc)] = cor
  cor_matrix[(n_strong + n_wai + 1):(n_strong + n_wai + n_wbc), 1] = cor
  
  if (!is.positive.definite(cor_matrix)) {
    return("The correlation matrix is not valid.")
  }
  
  # simulate the data from multivariate normal
  X = mvrnorm(n = n_sample, mu = rep(0, n_parameter), Sigma = cor_matrix) # var = 1, correlation = covariance
  
  beta = c(
    rep(coef_strong, n_strong),
    rep(bound/2, n_wai), 
    rep(bound/2, n_wbc),
    rep(0, n_null) 
  )
  
  Y = 1 + X %*% beta + rnorm(n_sample)
  data = as_tibble(data.frame(cbind(X, Y)))
  
  # Name the columns
  cols = c(
    str_c("strong", 1:n_strong, sep = "_"),
    str_c("wai", 1:n_wai, sep = "_"),
    str_c("wbc", 1:n_wbc, sep = "_"),
    str_c("null", 1:n_null, sep = "_"),
    "Y"
   )
   colnames(data) = cols
   data = data %>% 
     dplyr::select(Y, everything())

  return(data)
  
}

data_test = sim_data(1000)
```