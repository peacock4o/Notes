neural_networks
===============
- Category: Learning
- Tags: 
- Created: 2024-08-22T19:50:57-07:00

EPISODE 1
~~~~~~~~~

A neural network is made up of neurons
	"Neuron" being a thing that holds a value between 0 and 1
	The number inside the neuron is called an "activation." THink of it as being "lit up" that much.
Neurons are seperated into layers
	First layer - all the inputs you give it
	Last layer - all the possible outputs to predict from
		^^ This is for the classic number categorization example
		Each activation corresponds with how much the system thinks a given image is a certain number
	Hidden layers - layers in between first and last
		Number of neurons in hidden layers is somewhat arbitrary - experiment!
How do the layers behave? Why layers?
	Think about your own brain. (Still using the number example)
	When you see a number, how do you know what it is?
		Maybe you look for characteristics, like loops
		Maybe you determine those with certain edges in certain places
	Imagine each hidden layer is a subdivision of the number, starting with the smallest distinguishable edges and building up in each layer to make shapes
	In our number example, 
		the first layer is each pixel in the image
		The next layer is edges, defined by certain neurons in certain positions having certain values
		The next layer is shapes, defined by groupings of edges
		The last layer is numbers, defined by groupings of shapes
Weights
	Every neuron in a layer connects to each neuron in the layer next to it
	Each connection is given a weight. This is one object controlled by a parameter in the massive function that is the neural network.
Weight Math/Notation
	Imagine a neuron in a hidden layer.
	It receives all the values from the neurons in the previous layer, as expected.
		a1 + a2 + a3 + a4 + ... + an
		This doesn't add up to 1, you may say. We'll discuss that later.
	Each of these connections has a weight assocaited with it
		w1a1 + w2a2 + w3a3 + w4a4 + ... + wnan
	Then we squish this using a function. Commonly the sigmoid function (AKA logistic curve)
		sigma(w1a1 + w2a2 + w3a3 + w4a4 + ... + wnan)
		Think of this as "how positive" the weighted sum is
	What if you want the weighted sum to be activated only when it's over/under a certain amount?
		Called bias
		sigma(w1a1 + w2a2 + w3a3 + w4a4 + ... + wnan - bias)
		^ subtracting bias because we want the WS to be >10
	^^^ FINAL ACTIVATION FOR THAT NEURON! ^^^
"Learning"
	Think about the combinations for our numbers example
		784 neurons * 16 following + 16 neurons * 16 following + 16 neurons * 10 following + 16 + 16 + 10 biases = 
			13,002 PARAMETERS
	All "learning is" is tweaking the parameters to find the most optimal match for the training set
Matrix notation
	Note all activations in a layer as [a0^0 , a1^0 , a2^0 , a3^0 , a4^0 , ..., a5^0]
	Note all weights corresponding to neuron k as [wk,0 , wk,1 , wk,2 , wk,3 , wk,4 , ... wk,n]
	For each neuron k, the weighted sum is the matrix vector product of its weight vector with the previous layer's activation vector.
	We can represent the biases of all neurons in the "child" layer as [b0 , b1 , b2 , b3 , b4 , ... , bn]
		Add this vector to the previous matrix vector product
	Finally, apply sigmoid function to each value of the resulting vector after adding bias
Recap
	Really, every neuron has a function that outputs a value, not just a value

EPISODE 2
~~~~~~~~~

Training data
	Show the neural network a whole bunch of data, then have it adjust to best predict the shown values
	Data has "labels" of what the answer is supposed to be
Training process
	First, initialize weights and biases randomly
	Define a "cost function" - way for the computer to tell how "off" it was
		The value is small for correct classifications, large for being wrong
Optimizing lost cost value
	Remember that this model is essentially just a massive function
	Try to find the input where the cost function is minimized - negative slope!
		Can try to figure the value explicitely with a derivative
			Not always realistic with complex functions and large numebrs of parameters
		Can also find where slope bottoms out by moving left/right at a random point
			Also hard to figure out. Based on the random start point, the local minimum might not be the global minimum
			Local minimum = doable
			Global minimum = crazy hard
			Also note that as slope approaches 0, step size approaches 0
	What about for more than 1 input?
		Think about the cost graph as a mesh instead.
		Which direction (x, y) do I move in to decrease the cost fastest?
			Gradient (3d version of slope) tells us the fastest way to increase
			So, take the negative of this
	All "training" is is finding out the nudges that cause the cost to go down fastest
	It's important that the gradient is smooth so we can "ease into" a local minimum
		This is why ML neurons range between 0-1, while natural ones are binary
Performance
	It does pretty well - 96%
		99.97 is state-of-the-art
	Our model is only good at memorizing
		Can't "draw" a 5 or recognize uncertainty
			Flaws in training data being too perfect
	Newer models do this better.
		"Multilayer perceptron" was our example in the past 2 episodes
Learning Material
	Michael Nielsen website for book
