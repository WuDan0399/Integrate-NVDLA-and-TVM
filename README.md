# Integrate-NVDLA-and-TVM(Developing)

## Some notes 
1. Official code for NVDLA software [sw](https://github.com/nvdla/sw). Official document [sw doc](http://nvdla.org/sw/contents.html). But they currently go down (Sep 30 2020).
2.  To compile the umd part in sw, use command `make <path to sw>/sw/umd/out/core/src/compiler/libnvdla_compiler/libnvdla_compiler.so`
3. Main functions of compiler and parser are in `/umd/core/src/compiler/caffe/CaffeParser.cpp`
4. Parser output: A network. Defination of class network (in `/core/include/nvdla/INetwork.h`):

```CPP
class INetwork
{
public:
    virtual ITensor* addInput(const char * name, Dims4 dimensions) = 0;

    //	virtual void markChanged(const ILayer *) = 0;
    virtual bool markInput(ITensor * tensor) = 0;
    virtual void markOutput(ITensor * tensor) = 0;

    virtual IConvolutionLayer *    addConvolution   (ITensor * input, int numOutputs, int paddingValue, Dims2 kernelSize,
                                                    Dims2 tlPadding, Dims2 brPadding, Dims2 stride, Dims2 dilation,
                                                    Weights kernelWeights, Weights biasWeights, BiasMode biasMode, int numGroups) = 0;
    virtual IFullyConnectedLayer * addFullyConnected(ITensor * input, int outputSize, Weights kernelWeights, Weights biasWeights, BiasMode biasMode) = 0;
    virtual IActivationLayer *     addActivation    (ITensor * input, ActivationType type) = 0;
    virtual IPoolingLayer *        addPooling       (ITensor * input, PoolingType type,
                                                    Dims2 windowSize, Dims2 stride, Dims2 tlPadding, Dims2 brPadding) = 0;
    virtual ILRNLayer *            addLRN           (ITensor * input, int window, float alpha, float beta, float k) = 0;
    virtual IScaleLayer *          addScale         (ITensor * input, ScaleMode mode, Weights shift, Weights scale, Weights power) = 0;
    virtual IBatchNormLayer *      addBatchNorm     (ITensor * input, BatchNormMode mode, Weights mean, Weights variance, float epsilon) = 0;
    virtual ISoftMaxLayer *        addSoftMax       (ITensor*input) = 0;
    virtual IConcatenationLayer *  addConcatenation (ITensor*const*inputs, int numInputs) = 0;
    virtual ISliceLayer *          addSlice         (ITensor*input, int numOutputs) = 0;
    virtual IDeconvolutionLayer *  addDeconvolution (ITensor * input, int numOutputs, int paddingValue,
                                                    Dims2 kernelSize, Dims2 tlPadding, Dims2 brPadding, Dims2 stride, Dims2 dilation,
                                                    Weights kernelWeights, Weights biasWeights, BiasMode biasMode, int numGroups) = 0;
    virtual IElementWiseLayer   *  addElementWise   (ITensor *input0, ITensor* input1, ElementWiseOperation op) = 0;

    virtual int getNumInputs()  const  = 0;
    virtual int getNumOutputs() const  = 0;
    virtual int getNumLayers()  const  = 0;

    virtual ILayer  * getLayer(int index)  const = 0;
    virtual ITensor * getOutput(int index) const = 0;
    virtual ITensor * getInput(int index)  const = 0;


    class OutputDimensionsFormula
    {
    public:
        virtual Dims2 compute(Dims2 inputDims, Dims2 kernelSize,  Dims2 stride, Dims2 tlPadding, Dims2 brPadding, const char* layerName) const = 0;
        virtual Dims2 compute(Dims2 inputDims, Dims2 kernelSize,  Dims2 stride, Dims2 tlPadding, Dims2 brPadding, Dims2 dilation, const char* layerName) const = 0;
        virtual ~OutputDimensionsFormula() { }
    };

    class NetworkDefaultConvolutionFormula : public OutputDimensionsFormula
    {
    public:
        virtual Dims2 compute(Dims2 input, Dims2 kernel, Dims2 stride, Dims2 tlPadding, Dims2 brPadding, const char*) const;
        virtual Dims2 compute(Dims2 input, Dims2 kernel, Dims2 stride, Dims2 tlPadding, Dims2 brPadding, Dims2 dilation, const char*) const;
    };

    class NetworkDefaultDeconvolutionFormula : public OutputDimensionsFormula
    {
    public:
        virtual Dims2 compute(Dims2 input, Dims2 kernel, Dims2 stride, Dims2 tlPadding, Dims2 brPadding, const char*) const;
        virtual Dims2 compute(Dims2 input, Dims2 kernel, Dims2 stride, Dims2 tlPadding, Dims2 brPadding, Dims2 dilation, const char*) const;
    };

    class NetworkDefaultPoolingFormula : public OutputDimensionsFormula
    {
    public:
        virtual Dims2 compute(Dims2 input, Dims2 kernel, Dims2 stride, Dims2 tlPadding, Dims2 brPadding, const char*) const;
        virtual Dims2 compute(Dims2 /*input*/, Dims2 /*kernel*/, Dims2 /*stride*/, Dims2 /*tlPadding*/, Dims2 /*brPadding*/, Dims2 /*dilation*/, const char*) const
        {
            return Dims2(-1, -1);
        }
    };

    virtual void setPoolingOutputDimensionsFormula      (OutputDimensionsFormula* callback) = 0;
    virtual void setConvolutionOutputDimensionsFormula  (OutputDimensionsFormula* callback) = 0;
    virtual void setDeconvolutionOutputDimensionsFormula(OutputDimensionsFormula* callback) = 0;

    virtual OutputDimensionsFormula& getPoolingOutputDimensionsFormula()       const = 0;
    virtual OutputDimensionsFormula& getConvolutionOutputDimensionsFormula()   const = 0;
    virtual OutputDimensionsFormula& getDeconvolutionOutputDimensionsFormula() const = 0;

    virtual const std::vector<ITensor *> & getInputs()  const = 0;
    virtual const std::vector<ILayer * > & getLayers()  const = 0;
    virtual const std::vector<ITensor *> & getOutputs() const = 0;

protected:
    INetwork();
    virtual ~INetwork();

};
```
