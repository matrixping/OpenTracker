
USE_CAFFE=0
USE_CUDA=0
USE_BOOST=0
USE_SIMD=1
USE_GTEST=0 

CAFFE_PATH=/media/elab/sdd/mycodes/caffe

CC=gcc
CXX=g++
NVCC=nvcc

# used for cuda
ARCH= -gencode arch=compute_30,code=sm_30 \
      -gencode arch=compute_35,code=sm_35 \
      -gencode arch=compute_50,code=[sm_50,compute_50] \
      -gencode arch=compute_52,code=[sm_52,compute_52]

LDFLAGS= `pkg-config --libs opencv` -lstdc++ -lm 
# -O3 double the speed!!!!!!!!!
CXXFLAGS = -g -Wall `pkg-config --cflags opencv` -lstdc++ -lm -std=c++0x -msse4  -O3 

HEADERS = $(wildcard *.h)

TARGET_LIB = libeco.so # target lib

OBJS = ffttools.o fhog.o interpolator.o optimize_scores.o \
		regularization_filter.o feature_extractor.o feature_operator.o  \
		training.o sample_update.o eco.o metrics.o 

ifeq ($(USE_CAFFE), 1)
CXXFLAGS+= -DUSE_CAFFE
LDFLAGS+= -L$(CAFFE_PATH)/build/lib -lcaffe -lglog  -lprotobuf 
CXXFLAGS+= -I$(CAFFE_PATH)/build/include/ -I$(CAFFE_PATH)/include/ 
endif

ifeq ($(USE_CUDA), 1)
CXXFLAGS+= -DUSE_CUDA
LDFLAGS+= -L/usr/local/cuda/lib64 -lcuda -lcudart -lcublas -lcurand -lcudnn 
CXXFLAGS+= -I/usr/local/cuda/include/ 
endif

ifeq ($(USE_BOOST), 1)
CXXFLAGS+= -DUSE_BOOST
LDFLAGS+= -lboost_system -lboost_filesystem -lboost_regex
endif

ifeq ($(USE_SIMD), 1)
CXXFLAGS+= -DUSE_SIMD
HEADERS+= $(wildcard hog/*.hpp)
OBJS+= hog/convConst.o hog/gradientMex.o
endif

ifeq ($(USE_GTEST), 1)
GTEST_DIR = ../googletest
GTEST_HEADERS = $(GTEST_DIR)/include/gtest/*.h \
                $(GTEST_DIR)/include/gtest/internal/*.h
CXXFLAGS += -isystem $(GTEST_DIR)/include
endif

ifeq ($(USE_SIMD), 1)
	ALL+= makehog 
endif 
ALL+= runecotracker.bin
ifeq ($(USE_GTEST), 1) 
	ALL+= eco_unittest.bin 
endif

all: $(ALL)

runecotracker.bin: $(OBJS) runecotracker.o
	$(CC) -o $@ $^ $(LDFLAGS) 

$(TARGET_LIB): $(OBJS)
	$(CC) ${LDFLAGS} -shared -o $@ $^

%.o: %.c $(HEADERS)
	$(CC) -c -o $@ $< $(CFLAGS)

%.o: %.cc $(HEADERS)
	$(CXX) -c -o $@ $< $(CXXFLAGS)

%.o: %.cpp $(HEADERS)
	$(CXX) -c -o $@ $< $(CXXFLAGS)

%.o: %.cu $(HEADERS)
	$(NVCC) $(ARCH) --compiler-options "$(CXXFLAGS)" -c $< -o $@

ifeq ($(USE_SIMD), 1)
makehog:
	cd hog && make -j`nproc`
endif

ifeq ($(USE_GTEST), 1)
GTEST_SRCS_ = $(GTEST_DIR)/src/*.cc $(GTEST_DIR)/src/*.h $(GTEST_HEADERS)

gtest-all.o : $(GTEST_SRCS_)
	$(CXX) $(CXXFLAGS) -I$(GTEST_DIR) $(CXXFLAGS) -c \
            $(GTEST_DIR)/src/gtest-all.cc

gtest_main.o : $(GTEST_SRCS_)
	$(CXX) $(CXXFLAGS) -I$(GTEST_DIR) $(CXXFLAGS) -c \
            $(GTEST_DIR)/src/gtest_main.cc

gtest.a : gtest-all.o
	$(AR) $(ARFLAGS) $@ $^

gtest_main.a : gtest-all.o gtest_main.o
	$(AR) $(ARFLAGS) $@ $^
#-------------
eco_unittest.o : eco_unittest.cc ffttools.hpp $(GTEST_HEADERS)
	$(CXX) $(CXXFLAGS) -c eco_unittest.cc

eco_unittest.bin : ffttools.o eco_unittest.o gtest_main.a
	$(CXX) $(CXXFLAGS) $^ -o $@ $(LDFLAGS) -lpthread
endif

#cv gpu test========
cvgputest.o : cvgputest.cc 
	$(CXX) $(CXXFLAGS) -c cvgputest.cc

cvgputest.bin : cvgputest.o
	$(CXX) $(CXXFLAGS) $^ -o $@ $(LDFLAGS)

#==============
.PHONY: clean

clean:
	rm -rf ./.d *.o *.bin *.so *.a */*.o */*.bin
