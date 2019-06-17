set(FIPS_EXCEPTIONS ON)
set(FIPS_RTTI ON)

project(MPM LANGUAGES C CXX CUDA)

set(CMAKE_CXX_STANDARD 17)


if (NOT FIPS_IMPORT)
    cmake_minimum_required(VERSION 3.0)
    get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
    include("${FIPS_ROOT_DIR}/cmake/fips.cmake")
    fips_setup()
    fips_project(fips-gpumpm)
endif()


if ((FIPS_MACOS OR FIPS_WINDOWS OR FIPS_LINUX) AND NOT (FIPS_RASPBERRYPI OR FIPS_OSX_USE_ARC OR FIPS_UWP))

include(CheckLanguage)
check_language(CUDA)

find_package(CUDA REQUIRED)

fips_begin_lib(partio)
add_definitions(-DPARTIO_SE_ENABLED=0 -DPARTIO_USE_ZLIB)
fips_include_directories(./partio/src/lib)
fips_src("./partio/src/lib/io" RECURSE)
fips_src("./partio/src/lib/core" RECURSE)
fips_deps(zlib)
fips_end_lib()
set_target_properties(partio PROPERTIES OUTPUT_NAME partio POSITION_INDEPENDENT_CODE ON)

set(CMAKE_CUDA_FLAGS ${CMAKE_CUDA_FLAGS};-gencode arch=compute_61,code=sm_61)
message("arch=61")

fips_begin_lib(mpm-cpp)
fips_include_directories(./GPUMPM/src ./partio/src/lib)
fips_files(
    ./GPUMPM/src/Simulation/Material/Material.cpp
    ./GPUMPM/src/Simulation/Particle/ParticleDomain.cpp
    ./GPUMPM/src/Simulation/Grid/GridDomain.cpp
    ./GPUMPM/src/Simulation/DomainTransform/DomainTransformer.cpp
    ./GPUMPM/src/Simulation/MPM/Simulator.cpp
    ./GPUMPM/src/Simulation/MPM/SimulatorBuilder.cpp
    ./GPUMPM/src/MnBase/Math/Probability/Probability.cpp
)
fips_deps(zlib cudart cudadevrt partio)
fips_end_lib()

link_directories(/usr/local/cuda-10.1/lib64)
cuda_add_library(mpm_cuda
    ./GPUMPM/src/Simulation/MPM/Simulator.cu
    ./GPUMPM/src/Simulation/MPM/SimulatorKernels.cu
    ./GPUMPM/src/Simulation/Particle/ParticleDomain.cu
    ./GPUMPM/src/Simulation/Particle/ParticleKernels.cu
    ./GPUMPM/src/Simulation/DomainTransform/DomainTransformer.cu
    ./GPUMPM/src/Simulation/DomainTransform/DomainTransformKernels.cu
    ./GPUMPM/src/Simulation/Grid/GridDomain.cu
    ./GPUMPM/src/Simulation/Grid/GridKernels.cu
    ./GPUMPM/src/Simulation/TimeIntegrator/MPMTimeIntegrator.cu
    ./GPUMPM/src/Simulation/TimeIntegrator/ExplicitTimeIntegrator.cu
    ./GPUMPM/src/Simulation/TimeIntegrator/ImplicitTimeIntegrator.cu
    ./GPUMPM/src/Simulation/TimeIntegrator/P2GKernels.cu
    ./GPUMPM/src/Simulation/TimeIntegrator/GridUpdateKernels.cu
    ./GPUMPM/src/Simulation/TimeIntegrator/G2PKernels.cu
    ./GPUMPM/src/Simulation/TimeIntegrator/MPMComputationKernels.cu
    ./GPUMPM/src/Benchmarks.cu
    ./GPUMPM/src/MnBase/Math/Matrix/MatrixKernels.cu    
    ./GPUMPM/src/System/CudaDevice/CudaDeviceUtils.cu
    ./GPUMPM/src/System/CudaDevice/CudaDevice.cpp
    ./GPUMPM/src/SPGrid_Geometry.cpp
    ./GPUMPM/src/SPGrid_Utilities.cpp
    ./GPUMPM/src/System/CudaDevice/CudaHostUtils.hpp
    )
target_include_directories(mpm_cuda PRIVATE ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES})
target_include_directories(mpm_cuda PRIVATE ./GPUMPM/src ./partio/src/lib)
set_target_properties(mpm_cuda PROPERTIES CUDA_SEPERABLE_COMPILATION ON)
target_link_libraries(mpm_cuda mpm_cpp cudart cudadevrt partio)

endif()

if (NOT FIPS_IMPORT)
    fips_finish()
endif()