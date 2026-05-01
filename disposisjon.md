Introduction
    Smallsats have made HSI more available, what is HSI useful for? Large amounts of data that need to be handled
    Hypso at NTNU, monitoring of algal blooms, size of images and downlink speed
    Large images -> image compression
    What are challenges for implementing such compression
    FPGAs are the perfect fit! What are desirable characteristics of a good implementation?
    What is my contribution? Use of new weight update scheme to increase throughput of existing implementation, development of open source decompressor
    Layout of document

Background
    Introduction to CCSDS
        Issue 1 vs issue 2
        General overview, predictor + entropy encoder
        Predictor: Linear feedback prediction with adaptive weight updates
        Entropy encoder
    HSI introduction, band layout, terminology
        Capturing of HSIs, along track, sensor speed
    Present table with symbols
    Predictor 
        Present predictor block diagram
        Quantization of prediction residual
            Linear quantizers
            Quantizer index converted to mapped quantizer index, reversible process
        Quantizer index used to calculate a sample representative
            Needed to reproduce the prediction when actual samples are unavailable
            Present the steps needed to calculate sample representatives: high res, double res etc...
        Prediction neighborhood and local sums
            Prediction bands
            Narrow local sums
        Local differences + weights produce predicted central local difference
            Define predicted central local difference
            Define local difference vector
            Briefly discuss Reduced vs full prediction mode
            Define weight vector
            Weight updates
        Predicted sample value
            High res and double res
        Prediction residual
        Data dependencies
            Show dependency graph
            t-1 and z-1 deps
            Point out the most important ones
            Mention these may cause trouble for HW implementations
            Sample processing/storing orders
        Decompression
            Show block diagram
            Present formulas for reversal of quantizer index and prediction residual
            Selection of sample value from quantizer bin
    Header data
        Header data is prepended to the compressed bitstream
        Header is read from the compressed bitstream before image samples are reconstructed
    Entropy encoder 
        Brief example using Huffman encoding
        Discuss how the Hybrid encoder outperforms SA and BA for low bitrates
        Hybrid Encoder
            Code selection statistics
                Updates are done before encoding sample
            Low-entropy codes
                Encodes multiple samples using a single codeword
                Requires reverse decoding due codewords being output after multiple samples are read
                Suffix free rather than prefix free
            Variable to variable length codes
                Each codeword corresponds to a single sample
                Reversed goloumb codes to be suffix free
        Hybrid Decoder
            Image tail encodes final state of the code selection statistics, this is read first
            Decoding proceeds in reverse order
            Select entropy type based on statistics and decode
                Decode low-entropy codes
                Decode high-entropy codes
            After decoding, update code selection statistics
    Hardware accelerators 
        Can be used to speed up critical calculations drastically
        FPGAs
        FPGA vs ASIC
        Development methods for FPGA, VHDL vs HLS
            Verification: golden model, constrained random verification
        Multiprocessors and MPSoCs, zynq
        Different FPGAs have different speeds
        Parallelism: pipelining etc
    Image quality and coding performance 
        Compression ratio
        PSNR
    Compiled vs interpreted languages
        C++ and python

State-of-the-art
    What is the speed goal of implementations?
        Match speed of camera sensor to compress in real-time
    Implementations of issue 1
        They are fast
    Issue 2 presents a greater challenge due to data dependencies
    Hybrid encoder
        Fast implementations exist
        What is speed of Vorhaug implementation?
    First attempt: Barrios et al, serial, HLS, 18 MS/s
    Special consideration need to be made to accommodate the data dependencies
    FID ordering, Báscones et al.
        First real-time solution
        VHDL, deep pipeline
        Samples interleaving dependencies
        Reordering buffers
    Out-of-loop quantizer, Chatziantoniou et al. 
    These are pretty large 
    Mathematical optimization approach
        HLS? VHDL?
        Speculation
        Show conceptual pipeline
        Vorhaugs implementation
        Leaves weight update as critical path
    Skipping weight updates
        Not viable, too image dependant
    Delayed weight update scheme
    Table comparing of results the different approaches
        Delayed weight updates are allows higher throughput at low hardware penalty
    Software tools
        CNES
        Java implementation
        No open-source decompressor for hybrid encoder, should integrate with Vorhaugs implementation

Previous work
    Verification model
    Hardware implementation
        Present pipeline

Implementation and verification
    Verification model
        Purpose of the verification model
        C++ module inside python to increase execution time, pybind11
        Simple block diagram over the software
        Verification
            CWE test vectors
            Decompressed image references created using CNES decompressor
            Hypso 2, Aviris and Landsat
    Hardware
        Present new pipeline
        Weight update stage hardware block diagram
        Verification using random test data and image crops
        Test on flatsat
        Test on Hypso 2

Results
    Verification model
        Improvements in execution time
        Improvements in memory usage
    Delayed weight updates
        Reduction in compression performance
        PSNR
    Hardware
        Increase in clock frequency
        Only predictor
        New critical path
            Predictor only
            Full system
        Resource utilization
    Onboard test on Hypso 2

Discussion
    Open-source compressor/decompressor is nice, decompressor did not really exist before
        Should add support for SA and BA encoders
    Decompressor is slow, someone should make a faster one
    Checkout new critical paths? Would be nice to reach 100MHz
    Place in state-of-the-art
        Table comparing
        Speed results are comprable to those of the paper that suggested new weight update scheme
        This one is small, but at the cost of runtime reconfiguration
        Loss in compression performance is acceptable
        
