# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

ELM-TAM is a modified version of the Energy Exascale Earth System Model (E3SM) featuring the Trait-based Acclimation Model (TAM) integrated into the ELM (E3SM Land Model) component. This represents cutting-edge research in Earth system modeling with enhanced vegetation dynamics.

## Key Build and Development Commands

### Setting up a simulation case
The primary workflow uses CIME (Common Infrastructure for Modeling the Earth) build system:

1. **Create new case**: Use the template script `run_e3sm.template.sh` as starting point
2. **CIME workflow**:
   ```bash
   # From cime/scripts directory (if available):
   ./create_newcase --case CASENAME --compset COMPSET --res RESOLUTION --machine MACHINE
   cd CASENAME
   ./case.setup
   ./case.build
   ./case.submit
   ```

3. **Key script**: `run_e3sm.template.sh` - Copy and modify this template for your simulations
   - Set MACHINE, PROJECT, COMPSET, RESOLUTION, CASE_NAME
   - Configure build and run options
   - Includes pre-configured namelist settings for atmosphere (EAM) and land (ELM) components

### Development workflow
- **Fortran compilation**: Uses CMake build system with component-specific Makefiles
- **Testing**: No standardized test commands identified - check component-specific directories for tests
- **Code modification**: TAM features are conditionally compiled using `#if defined(TAM)` preprocessor directives

## Code Architecture

### Multi-component Earth system model structure:
- **components/eam/**: Energy Atmosphere Model - atmospheric physics and chemistry
- **components/elm/**: E3SM Land Model - land surface processes **with TAM modifications**
- **components/mpas-ocean/**: MPAS Ocean component - ocean dynamics
- **components/mpas-seaice/**: MPAS Sea Ice component
- **components/cice/**: Community Ice CodE - sea ice dynamics
- **components/mosart/**: River transport model
- **components/mpas-albany-landice/**: Land ice dynamics

### TAM Integration Points
The Trait-based Acclimation Model (TAM) is integrated primarily in ELM with conditional compilation:
- **TAM-specific code**: Located throughout `components/elm/src/` with `#if defined(TAM)` guards
- **Key modified files**:
  - `components/elm/src/main/pftvarcon.F90` - Plant functional type parameters
  - `components/elm/src/data_types/VegetationDataType.F90` - Extended vegetation state variables
  - `components/elm/src/biogeochem/` modules - Enhanced biogeochemical processes
  - `components/elm/src/dyn_subgrid/` modules - Dynamic land use with TAM traits

### Build system components:
- **CIME infrastructure**: `cime_config/` - machine configurations, component settings
- **CMake system**: `components/cmake/` - build configuration for each component
- **Driver coupling**: `driver-mct/` - Model Coupling Toolkit for component interactions

### Key directories for development:
- **Component source**: `components/[component]/src/` - Core model physics
- **CIME config**: `components/[component]/cime_config/` - Build and namelist configuration
- **Shared utilities**: `share/` - Common utilities, timing, data structures

## TAM-Specific Development Notes

### Conditional compilation
- TAM features use `#if defined(TAM)` preprocessor directives
- Affects root carbon allocation, vegetation dynamics, and trait representation
- Found in ~38 files across the ELM component

### Key TAM modifications:
- Enhanced fine root representation (transport, active, metabolic components)
- Trait-based acclimation for vegetation responses
- Modified carbon-nitrogen-phosphorus cycling with trait dependencies

### Working with TAM code:
- Current branch: `TAM` (feature branch from master)
- When modifying TAM features, ensure conditional compilation is maintained
- Test both TAM and non-TAM configurations when possible

## Important Configuration Files

- **run_e3sm.template.sh**: Main simulation setup template with preconfigured namelists
- **cime_config/machines/**: Machine-specific build configurations
- **components/[comp]/cime_config/config_component.xml**: Component configuration definitions
- **User namelist templates**: `components/[comp]/cime_config/user_nl_[comp]`

## Development Best Practices

- Follow E3SM contribution guidelines in CONTRIBUTING.md
- Maintain compatibility with base E3SM when possible
- Use proper Fortran coding standards consistent with existing codebase
- Document TAM-specific modifications clearly with conditional compilation guards
- Test on supported HPC systems listed in machine configurations

## Repository Maintenance

- **Git management**: This is a fork focused on TAM development
- **Upstream sync**: Coordinate with base E3SM repository for major updates
- **Branch strategy**: `TAM` branch for feature development, `master` tracks base E3SM