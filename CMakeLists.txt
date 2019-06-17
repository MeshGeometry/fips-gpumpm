set(FIPS_EXCEPTIONS ON)
set(FIPS_RTTI ON)

project(MPM LANGUAGES C CXX CUDA)


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
fips_src("/partio/lib" GROUP_FOLDERS)
fips_deps(zlib)
fips_end_lib()

fips_begin_lib(mpm-cpp)
fips_dirs("/GPUMPM/src")
fips_files(
    /Simulation/Material/Material.cpp
    /Simulation/Particle/ParticleDomain.cpp
    /Simulation/Grid/GridDomain.cpp
    /Simulation/DomainTransform/DomainTransformer.cpp
    /Simulation/MPM/Simulator.cpp
    /Simulation/MPM/SimulatorBuilder.cpp
    /MnBase/Math/Probability/Probability.cpp
)
fips_deps(zlib cudart cudadevrt partio)
fips_end_lib()

cuda_add_library(mpm_cuda
    /GPUMPM/src/Simulation/MPM/Simulator.cu
    /GPUMPM/src/Simulation/MPM/SimulatorKernels.cu
    /GPUMPM/src/Simulation/Particle/ParticleDomain.cu
    /GPUMPM/src/Simulation/Particle/ParticleKernels.cu
    /GPUMPM/src/Simulation/DomainTransform/DomainTransformer.cu
    /GPUMPM/src/Simulation/DomainTransform/DomainTransformKernels.cu
    /GPUMPM/src/Simulation/Grid/GridDomain.cu
    /GPUMPM/src/Simulation/Grid/GridKernels.cu
    /GPUMPM/src/Simulation/TimeIntegrator/MPMTimeIntegrator.cu
    /GPUMPM/src/Simulation/TimeIntegrator/ExplicitTimeIntegrator.cu
    /GPUMPM/src/Simulation/TimeIntegrator/ImplicitTimeIntegrator.cu
    /GPUMPM/src/Simulation/TimeIntegrator/P2GKernels.cu
    /GPUMPM/src/Simulation/TimeIntegrator/GridUpdateKernels.cu
    /GPUMPM/src/Simulation/TimeIntegrator/G2PKernels.cu
    /GPUMPM/src/Simulation/TimeIntegrator/MPMComputationKernels.cu
    /GPUMPM/src/Benchmarks.cu
    /GPUMPM/src/MnBase/Math/Matrix/MatrixKernels.cu    
    /GPUMPM/src/System/CudaDevice/CudaDeviceUtils.cu
    /GPUMPM/src/System/CudaDevice/CudaDevice.cpp
    /GPUMPM/src/SPGrid_Geometry.cpp
    /GPUMPM/src/SPGrid_Utilities.cpp
    /GPUMPM/src/System/CudaDevice/CudaHostUtils.hpp
    )
target_include_directories()
target_link_libraries(mpm_cuda mpm_cpp cudart cudadevrt partio)

endif()

if (NOT FIPS_IMPORT)
    fips_finish()
endif()