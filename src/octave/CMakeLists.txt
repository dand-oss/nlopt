set (MFILES NLOPT_GN_DIRECT.m NLOPT_GN_DIRECT_L.m NLOPT_GN_DIRECT_L_RAND.m NLOPT_GN_DIRECT_NOSCAL.m NLOPT_GN_DIRECT_L_NOSCAL.m NLOPT_GN_DIRECT_L_RAND_NOSCAL.m NLOPT_GN_ORIG_DIRECT.m NLOPT_GN_ORIG_DIRECT_L.m NLOPT_GD_STOGO.m NLOPT_GD_STOGO_RAND.m NLOPT_LD_LBFGS.m NLOPT_LN_PRAXIS.m NLOPT_LD_VAR1.m NLOPT_LD_VAR2.m NLOPT_LD_TNEWTON.m NLOPT_LD_TNEWTON_RESTART.m NLOPT_LD_TNEWTON_PRECOND.m NLOPT_LD_TNEWTON_PRECOND_RESTART.m NLOPT_GN_CRS2_LM.m NLOPT_GN_MLSL.m NLOPT_GD_MLSL.m NLOPT_GN_MLSL_LDS.m NLOPT_GD_MLSL_LDS.m NLOPT_LD_MMA.m NLOPT_LN_COBYLA.m NLOPT_LN_NEWUOA.m NLOPT_LN_NEWUOA_BOUND.m NLOPT_LN_NELDERMEAD.m NLOPT_LN_SBPLX.m NLOPT_LN_AUGLAG.m NLOPT_LD_AUGLAG.m NLOPT_LN_AUGLAG_EQ.m NLOPT_LD_AUGLAG_EQ.m NLOPT_LN_BOBYQA.m NLOPT_GN_ISRES.m NLOPT_AUGLAG.m NLOPT_AUGLAG_EQ.m NLOPT_G_MLSL.m NLOPT_G_MLSL_LDS.m NLOPT_LD_SLSQP.m NLOPT_LD_CCSAQ.m NLOPT_GN_ESCH.m NLOPT_GN_AGS.m)
set (M_DATA ${MFILES} nlopt_minimize.m nlopt_minimize_constrained.m)


if (Matlab_FOUND AND Matlab_MX_LIBRARY)
  matlab_add_mex (NAME nlopt_optimize-mex SRC nlopt_optimize-mex.c OUTPUT_NAME nlopt_optimize LINK_TO ${nlopt_lib})
  target_include_directories (nlopt_optimize-mex PRIVATE ${PROJECT_SOURCE_DIR}/src/api)

  if (NLOPT_CXX)
    set_target_properties (nlopt_optimize-mex PROPERTIES LINKER_LANGUAGE CXX)
  endif ()

  set (NLOPT_INSTALL_MEXDIR ${NLOPT_INSTALL_LIBDIR}/matlab CACHE STRING "mex dir")
  install (TARGETS nlopt_optimize-mex DESTINATION ${NLOPT_INSTALL_MEXDIR})

  set (NLOPT_INSTALL_MDIR ${NLOPT_INSTALL_LIBDIR}/matlab CACHE STRING "m dir")
  install (FILES ${M_DATA} DESTINATION ${NLOPT_INSTALL_MDIR})
endif ()



if (OCTAVE_FOUND)
  file (WRITE ${CMAKE_CURRENT_BINARY_DIR}/nlopt_optimize_usage.h "#define NLOPT_OPTIMIZE_USAGE \\\n")
  file (STRINGS ${CMAKE_CURRENT_SOURCE_DIR}/nlopt_optimize.m INPUT_LINES)
  foreach (INPUT_LINE ${INPUT_LINES})
    string (REGEX REPLACE "^% " "" INPUT_LINE ${INPUT_LINE})
    string (REGEX REPLACE "^%" "" INPUT_LINE ${INPUT_LINE})
    foreach (repl_expr "tolerance" "help NLOPT_LN_SBPLX" "population")
      string (REGEX REPLACE "\"${repl_expr}\"" "${repl_expr}" INPUT_LINE "${INPUT_LINE}")
    endforeach ()
    file (APPEND ${CMAKE_CURRENT_BINARY_DIR}/nlopt_optimize_usage.h "\"${INPUT_LINE}\\n\" \\\n")
  endforeach ()
  file (APPEND ${CMAKE_CURRENT_BINARY_DIR}/nlopt_optimize_usage.h "\n")

  octave_add_oct (nlopt_optimize SOURCES nlopt_optimize-oct.cc ${CMAKE_CURRENT_BINARY_DIR}/nlopt_optimize_usage.h LINK_LIBRARIES ${nlopt_lib})
  target_include_directories (nlopt_optimize PRIVATE ${OCTAVE_INCLUDE_DIRS} ${CMAKE_CURRENT_BINARY_DIR} ${PROJECT_SOURCE_DIR}/src/api)

  if (OCTAVE_VERSION_MAJOR GREATER 10)
    target_compile_features(nlopt_optimize PRIVATE cxx_std_17)
  endif()

  if (NOT DEFINED INSTALL_OCT_DIR)
    file (RELATIVE_PATH INSTALL_OCT_DIR ${OCTAVE_ROOT_DIR} ${OCTAVE_OCT_SITE_DIR})
  endif ()
  install (TARGETS nlopt_optimize DESTINATION ${INSTALL_OCT_DIR})

  if (NOT DEFINED INSTALL_M_DIR)
    file (RELATIVE_PATH INSTALL_M_DIR ${OCTAVE_ROOT_DIR} ${OCTAVE_M_SITE_DIR})
  endif ()

  install (FILES ${M_DATA} DESTINATION ${INSTALL_M_DIR})
endif ()
