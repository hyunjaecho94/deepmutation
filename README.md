# deepmutation
original idea: https://arxiv.org/pdf/1805.05206.pdf

This is a project I selected as my course project for Analysis of Software Artifacts in Fall 2019 at the University of Virginia. As explained in the above paper, the goal is insert mutants into the model and the data in order to evaluate the quality of the testing data in deep neural networks. Below is my proposal for the project, which largely overlaps with the original paper, DeepMutation: Mutation Testing of Deep Learning Systems.

----------------------------------------------------------------------------------------------------------------

The workflow of deep learning software development roughly respects the following procedure:
obtain a suitable dataset, preprocess, split it into training and testing data sets, develop a model, train it on the training data, evaluate the performance of the model on the test data before deployment. Since the test data is the main source to evaluate the model, its quality is of critical importance. Therefore, in this project, I ask and will try to answer the following question: “How can we evaluate the quality of the test dataset in deep learning”?

To answer this question, I use mutation testing and generally follow the procedure of [Ma et al](https://arxiv.org/pdf/1805.05206.pdf):
 	
	1. Train a DNN model on the original training data. Specifically, I will use a convolutional neural network as my model Moriginal and the MNIST data set.
    2. Test the neural network on the test data. Mark the test data points that are correctly classified by the model. Call it <a href="https://www.codecogs.com/eqnedit.php?latex=D_{correct}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D_{correct}" title="D_{correct}" /></a>.
    3. Insert a mutant. Example of mutants:
              1. removing, repeating, mislabeling the training data
              2. removing, adding hidden layers in DNN
	      3. initializing the neuron weights with pre-trained weights
	      4. negating and removing the activation function
    4. For each mutant <a href="https://www.codecogs.com/eqnedit.php?latex=m^k" target="_blank"><img src="https://latex.codecogs.com/gif.latex?m^k" title="m^k" /></a>, I will re-evaluate the <a href="https://www.codecogs.com/eqnedit.php?latex=m^k" target="_blank"><img src="https://latex.codecogs.com/gif.latex?m^k" title="m^k" /></a>-mutated DNN on <a href="https://www.codecogs.com/eqnedit.php?latex=D_{correct}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D_{correct}" title="D_{correct}" /></a>. If any of <a href="https://www.codecogs.com/eqnedit.php?latex=D_{correct}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D_{correct}" title="D_{correct}" /></a> is misclassified, m<a href="https://www.codecogs.com/eqnedit.php?latex=m^k" target="_blank"><img src="https://latex.codecogs.com/gif.latex?m^k" title="m^k" /></a>k survives and is evaluated. Otherwise, it is killed (no effect). The evaluation method of <a href="https://www.codecogs.com/eqnedit.php?latex=m^k" target="_blank"><img src="https://latex.codecogs.com/gif.latex?m^k" title="m^k" /></a> is provided below.

After testing out the results on the MNIST data set, I will try mutation testing on more complex data sets, such as CIFAR10 or MS-COCO. In addition, I will discover new, effective mutants in addition to the ones mentioned in step 3. For example, I believe that swapping the order of hidden layers will be a very useful mutant. For example, if the original two hidden layers contained x and y neurons respectively, I would change the order to y and x. Often, DL engineers design networks based on previous experiences on similar data sets or empirically on how different layer ordering performs on training/validation data sets. Being able to draw a conclusion on layer ordering based on the results from mutation testing presents another way for DL engineers to design more effective DNNs.

It is safe to believe that the test data points that get misclassified after mutants are inserted are of lower quality than those unaffected by mutation. The term quality in the context of classification refers to how far a data point lies from the model’s decision boundary. The farther it is, the more certain the model is about to which class it belongs to—therefore, the higher its quality.

Suppose that the number of different classes is c; each data point belongs to one of {class1, class2, … classc}. I evaluate the mutation score for mutant <a href="https://www.codecogs.com/eqnedit.php?latex=m^k" target="_blank"><img src="https://latex.codecogs.com/gif.latex?m^k" title="m^k" /></a> as follows:

<a href="https://www.codecogs.com/eqnedit.php?latex=MutationScore(D_{correct},m^k)&space;=&space;\sum_{i=1}^{c}&space;\frac{count_{misclassify}(D_{correct},m^k,class_i)}{|D_{correct}&space;\in&space;class_i&space;|}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?MutationScore(D_{correct},m^k)&space;=&space;\sum_{i=1}^{c}&space;\frac{count_{misclassify}(D_{correct},m^k,class_i)}{|D_{correct}&space;\in&space;class_i&space;|}" title="MutationScore(D_{correct},m^k) = \sum_{i=1}^{c} \frac{count_misclassify(D_{correct},m^k,class_i)}{|D_{correct} \in class_i |}" /></a>

where count_misclassify(<a href="https://www.codecogs.com/eqnedit.php?latex=D_{correct}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D_{correct}" title="D_{correct}" /></a>, mk, classi) is the number of test data points in <a href="https://www.codecogs.com/eqnedit.php?latex=D_{correct}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D_{correct}" title="D_{correct}" /></a> that belong to <a href="https://www.codecogs.com/eqnedit.php?latex=class_i" target="_blank"><img src="https://latex.codecogs.com/gif.latex?class_i" title="class_i" /></a>  that <a href="https://www.codecogs.com/eqnedit.php?latex=m^k" target="_blank"><img src="https://latex.codecogs.com/gif.latex?m^k" title="m^k" /></a> causes the model to misclassify. I divide this value by the number of test data points in <a href="https://www.codecogs.com/eqnedit.php?latex=D_{correct}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D_{correct}" title="D_{correct}" /></a> that belong to <a href="https://www.codecogs.com/eqnedit.php?latex=class_i" target="_blank"><img src="https://latex.codecogs.com/gif.latex?class_i" title="class_i" /></a> (<a href="https://www.codecogs.com/eqnedit.php?latex=|D_{correct}&space;\in&space;class_i&space;|" target="_blank"><img src="https://latex.codecogs.com/gif.latex?|D_{correct}&space;\in&space;class_i&space;|" title="|D_{correct} \in class_i |" /></a>). This way of calculating the mutation score handles the case where two data sets whose sizes are largely different and also differentiate the mutant that causes only a few of <a href="https://www.codecogs.com/eqnedit.php?latex=D_{correct}&space;\in&space;class_i" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D_{correct}&space;\in&space;class_i" title="D_{correct} \in class_i" /></a> to be misclassified from the one that causes the majority of them to be.

It is highly likely, that most, if not every, mutant will result in different mutation scores. One way to interpret this is how detrimentally a mutant affects the model’s performance: the higher the score, the more detrimental its effect. This project will matter to DL developers who build DNNs to decide on the quality of the test data, provide them a way to evaluate data preprocessing methods, gain confidence on their model’s performance on the test data, as well as those who gather the data itself.
