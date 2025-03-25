## Mathematical Representation of the System
Define the system using functions and composition, emphasizing how Providers and Adaptors work together.

### 1. Providers
A Provider generates inference data based on some input. We can define it as a function:

\[P: I \rightarrow O_P\]

where:
- \(I\) is the input domain (e.g., raw data or a query)
- \(O_P\) is the output domain of the Provider, representing the inference data it produces

For example, if you have two Providers, \(P_1\) and \(P_2\):
- \(O_{P_1} = P_1(I)\)
- \(O_{P_2} = P_2(I)\)

This mathematical representation shows that:
1. Each Provider is a function that transforms input data into inference results
2. Different Providers can produce different types of outputs from the same input
3. The input domain can be standardized across Providers while output domains may vary

#### Example Use Cases:
1. Text Classification Provider:
   - Input: Text string
   - Output: Category label and confidence score
   
2. Price Prediction Provider:
   - Input: Historical price data
   - Output: Predicted price value

3. Sentiment Analysis Provider:
   - Input: Social media text
   - Output: Sentiment score and polarity

1. Adaptors
An Adaptor transforms inference data from one format to another. We can define it as a function:



