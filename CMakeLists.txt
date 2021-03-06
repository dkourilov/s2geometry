cmake_minimum_required(VERSION 2.8.12)
project(s2-geometry)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
include(CheckCXXCompilerFlag)
include(FindPackageHandleStandardArgs)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/third_party/cmake")

find_package(GFlags REQUIRED)
find_package(Glog REQUIRED)
find_package(OpenSSL REQUIRED)
# pthreads isn't used directly, but this is still required for std::thread.
find_package(Threads REQUIRED)
find_package(SWIG)
find_package(PythonInterp)
find_package(PythonLibs)

# FindGFlags.cmake and FindGlog.cmake do not seem to implement REQUIRED;
# check manually.
if (NOT ${GFLAGS_FOUND})
    message(FATAL_ERROR "GFLags required but not found")
endif()
if (NOT ${GLOG_FOUND})
    message(FATAL_ERROR "Glog required but not found")
endif()

# Avoid "Policy CMP0042 is not set" warning on macOS.
if (APPLE)
    set(CMAKE_MACOSX_RPATH TRUE)
endif()

add_definitions(-Wno-deprecated-declarations -std=c++11)

# If OpenSSL is installed in a non-standard location, configure with
# something like:
# OPENSSL_ROOT_DIR=/usr/local/opt/openssl cmake ..
include_directories(
    ${GFLAGS_INCLUDE_DIRS} ${GLOG_INCLUDE_DIRS} ${OPENSSL_INCLUDE_DIR}
    ${PYTHON_INCLUDE_DIRS})
include_directories(src)

add_library(s2 SHARED
            src/s2/base/stringprintf.cc
            src/s2/base/strtoint.cc
            src/s2/id_set_lexicon.cc
            src/s2/mutable_s2shapeindex.cc
            src/s2/r2rect.cc
            src/s2/s1angle.cc
            src/s2/s1chordangle.cc
            src/s2/s1interval.cc
            src/s2/s2boolean_operation.cc
            src/s2/s2builder.cc
            src/s2/s2builder_graph.cc
            src/s2/s2builderutil_closed_set_normalizer.cc
            src/s2/s2builderutil_find_polygon_degeneracies.cc
            src/s2/s2builderutil_s2point_vector_layer.cc
            src/s2/s2builderutil_s2polygon_layer.cc
            src/s2/s2builderutil_s2polyline_layer.cc
            src/s2/s2builderutil_s2polyline_vector_layer.cc
            src/s2/s2builderutil_snap_functions.cc
            src/s2/s2builderutil_testing.cc
            src/s2/s2cap.cc
            src/s2/s2cell.cc
            src/s2/s2cellid.cc
            src/s2/s2cellunion.cc
            src/s2/s2centroids.cc
            src/s2/s2closestedgequery.cc
            src/s2/s2closestpointquery.cc
            src/s2/s2contains_vertex_query.cc
            src/s2/s2convexhullquery.cc
            src/s2/s2coords.cc
            src/s2/s2crossingedgequery.cc
            src/s2/s2debug.cc
            src/s2/s2earth.cc
            src/s2/s2edge_clipping.cc
            src/s2/s2edge_crosser.cc
            src/s2/s2edge_crossings.cc
            src/s2/s2edge_distances.cc
            src/s2/s2edge_tessellator.cc
            src/s2/s2error.cc
            src/s2/s2latlng.cc
            src/s2/s2latlngrect.cc
            src/s2/s2latlngrect_bounder.cc
            src/s2/s2lax_loop_shape.cc
            src/s2/s2lax_polygon_shape.cc
            src/s2/s2lax_polyline_shape.cc
            src/s2/s2loop.cc
            src/s2/s2measures.cc
            src/s2/s2metrics.cc
            src/s2/s2min_distance_targets.cc
            src/s2/s2paddedcell.cc
            src/s2/s2pointcompression.cc
            src/s2/s2pointregion.cc
            src/s2/s2pointutil.cc
            src/s2/s2polygon.cc
            src/s2/s2polyline.cc
            src/s2/s2polyline_simplifier.cc
            src/s2/s2predicates.cc
            src/s2/s2projections.cc
            src/s2/s2r2rect.cc
            src/s2/s2region.cc
            src/s2/s2region_term_indexer.cc
            src/s2/s2regioncoverer.cc
            src/s2/s2regionintersection.cc
            src/s2/s2regionunion.cc
            src/s2/s2shapeindex.cc
            src/s2/s2shapeindex_buffered_region.cc
            src/s2/s2shapeutil.cc
            src/s2/s2shapeutil_build_polygon_boundaries.cc
            src/s2/s2shapeutil_contains_brute_force.cc
            src/s2/s2shapeutil_edge_iterator.cc
            src/s2/s2shapeutil_get_reference_point.cc
            src/s2/s2shapeutil_range_iterator.cc
            src/s2/s2textformat.cc
            src/s2/s2wedge_relations.cc
            src/s2/strings/serialize.cc
            src/s2/third_party/absl/base/dynamic_annotations.cc
            src/s2/third_party/absl/base/internal/log_severity.cc
            src/s2/third_party/absl/base/internal/raw_logging.cc
            src/s2/third_party/absl/base/internal/throw_delegate.cc
            src/s2/third_party/absl/numeric/int128.cc
            src/s2/third_party/absl/strings/ascii.cc
            src/s2/third_party/absl/strings/match.cc
            src/s2/third_party/absl/strings/numbers.cc
            src/s2/third_party/absl/strings/str_cat.cc
            src/s2/third_party/absl/strings/str_split.cc
            src/s2/third_party/absl/strings/string_view.cc
            src/s2/third_party/absl/strings/strip.cc
            src/s2/third_party/absl/strings/internal/memutil.cc
            src/s2/util/bits/bit-interleave.cc
            src/s2/util/bits/bits.cc
            src/s2/util/coding/coder.cc
            src/s2/util/coding/varint.cc
            src/s2/util/math/exactfloat/exactfloat.cc
            src/s2/util/math/mathutil.cc
            src/s2/util/units/length-units.cc)
add_library(s2testing STATIC src/s2/s2testing.cc)
target_link_libraries(
    s2
    ${GFLAGS_LIBRARIES} ${GLOG_LIBRARIES} ${OPENSSL_LIBRARIES}
    ${CMAKE_THREAD_LIBS_INIT})

# We don't need to install all headers, only those
# transitively included by s2 headers we are exporting.
install(FILES src/s2/_fpcontractoff.h
              src/s2/id_set_lexicon.h
              src/s2/mutable_s2shapeindex.h
              src/s2/r1interval.h
              src/s2/r2.h
              src/s2/r2rect.h
              src/s2/s1angle.h
              src/s2/s1chordangle.h
              src/s2/s1interval.h
              src/s2/s2boolean_operation.h
              src/s2/s2builder.h
              src/s2/s2builder_graph.h
              src/s2/s2builder_layer.h
              src/s2/s2builderutil_closed_set_normalizer.h
              src/s2/s2builderutil_find_polygon_degeneracies.h
              src/s2/s2builderutil_s2point_vector_layer.h
              src/s2/s2builderutil_s2polygon_layer.h
              src/s2/s2builderutil_s2polyline_layer.h
              src/s2/s2builderutil_s2polyline_vector_layer.h
              src/s2/s2builderutil_snap_functions.h
              src/s2/s2builderutil_testing.h
              src/s2/s2cap.h
              src/s2/s2cell.h
              src/s2/s2cellid.h
              src/s2/s2cellunion.h
              src/s2/s2centroids.h
              src/s2/s2closestedgequery.h
              src/s2/s2closestedgequery_base.h
              src/s2/s2closestpointquery.h
              src/s2/s2closestpointquery_base.h
              src/s2/s2contains_point_query.h
              src/s2/s2contains_vertex_query.h
              src/s2/s2convexhullquery.h
              src/s2/s2coords_internal.h
              src/s2/s2coords.h
              src/s2/s2crossingedgequery.h
              src/s2/s2debug.h
              src/s2/s2distance_target.h
              src/s2/s2earth.h
              src/s2/s2edge_clipping.h
              src/s2/s2edge_crosser.h
              src/s2/s2edge_crossings_internal.h
              src/s2/s2edge_crossings.h
              src/s2/s2edge_distances.h
              src/s2/s2edge_tessellator.h
              src/s2/s2edge_vector_shape.h
              src/s2/s2error.h
              src/s2/s2latlng.h
              src/s2/s2latlngrect.h
              src/s2/s2latlngrect_bounder.h
              src/s2/s2lax_loop_shape.h
              src/s2/s2lax_polygon_shape.h
              src/s2/s2lax_polyline_shape.h
              src/s2/s2loop.h
              src/s2/s2measures.h
              src/s2/s2metrics.h
              src/s2/s2min_distance_targets.h
              src/s2/s2paddedcell.h
              src/s2/s2point.h
              src/s2/s2point_vector_shape.h
              src/s2/s2pointcompression.h
              src/s2/s2pointindex.h
              src/s2/s2pointregion.h
              src/s2/s2pointutil.h
              src/s2/s2polygon.h
              src/s2/s2polyline.h
              src/s2/s2polyline_simplifier.h
              src/s2/s2predicates_internal.h
              src/s2/s2predicates.h
              src/s2/s2projections.h
              src/s2/s2r2rect.h
              src/s2/s2region.h
              src/s2/s2region_term_indexer.h
              src/s2/s2regioncoverer.h
              src/s2/s2regionintersection.h
              src/s2/s2regionunion.h
              src/s2/s2shape.h
              src/s2/s2shapeindex.h
              src/s2/s2shapeindex_buffered_region.h
              src/s2/s2shapeindex_region.h
              src/s2/s2shapeutil.h
              src/s2/s2shapeutil_build_polygon_boundaries.h
              src/s2/s2shapeutil_contains_brute_force.h
              src/s2/s2shapeutil_count_edges.h
              src/s2/s2shapeutil_edge_iterator.h
              src/s2/s2shapeutil_get_reference_point.h
              src/s2/s2shapeutil_range_iterator.h
              src/s2/s2shapeutil_shape_edge.h
              src/s2/s2shapeutil_shape_edge_id.h
              src/s2/s2testing.h
              src/s2/s2textformat.h
              src/s2/s2wedge_relations.h
              src/s2/sequence_lexicon.h
              src/s2/value_lexicon.h
        DESTINATION include/s2)
install(FILES src/s2/base/casts.h
              src/s2/base/mutex.h
              src/s2/base/port.h
              src/s2/base/spinlock.h
        DESTINATION include/s2/base)
install(FILES src/s2/third_party/absl/algorithm/algorithm.h
        DESTINATION include/s2/third_party/absl/algorithm)
install(FILES src/s2/third_party/absl/base/attributes.h
              src/s2/third_party/absl/base/casts.h
              src/s2/third_party/absl/base/config.h
              src/s2/third_party/absl/base/dynamic_annotations.h
              src/s2/third_party/absl/base/integral_types.h
              src/s2/third_party/absl/base/macros.h
              src/s2/third_party/absl/base/optimization.h
              src/s2/third_party/absl/base/policy_checks.h
              src/s2/third_party/absl/base/port.h
              src/s2/third_party/absl/base/thread_annotations.h
        DESTINATION include/s2/third_party/absl/base)
install(FILES src/s2/third_party/absl/base/internal/identity.h
              src/s2/third_party/absl/base/internal/throw_delegate.h
              src/s2/third_party/absl/base/internal/unaligned_access.h
        DESTINATION include/s2/third_party/absl/base/internal)
install(FILES src/s2/third_party/absl/container/inlined_vector.h
        DESTINATION include/s2/third_party/absl/container)
install(FILES src/s2/third_party/absl/memory/memory.h
        DESTINATION include/s2/third_party/absl/memory)
install(FILES src/s2/third_party/absl/meta/type_traits.h
        DESTINATION include/s2/third_party/absl/meta)
install(FILES src/s2/third_party/absl/numeric/int128.h
              src/s2/third_party/absl/numeric/int128_have_intrinsic.inc
              src/s2/third_party/absl/numeric/int128_no_intrinsic.inc
        DESTINATION include/s2/third_party/absl/numeric)
install(FILES src/s2/third_party/absl/strings/string_view.h
        DESTINATION include/s2/third_party/absl/strings)
install(FILES src/s2/third_party/absl/types/span.h
        DESTINATION include/s2/third_party/absl/types)
install(FILES src/s2/util/bits/bits.h
        DESTINATION include/s2/util/bits)
install(FILES src/s2/util/btree/btree.h
              src/s2/util/btree/btree_container.h
              src/s2/util/btree/btree_map.h
              src/s2/util/btree/btree_set.h
        DESTINATION include/s2/util/btree)
install(FILES src/s2/util/coding/coder.h
              src/s2/util/coding/varint.h
        DESTINATION include/s2/util/coding)
install(FILES src/s2/util/endian/endian.h
        DESTINATION include/s2/util/endian)
install(FILES src/s2/util/gtl/compact_array.h
              src/s2/util/gtl/container_logging.h
              src/s2/util/gtl/dense_hash_set.h
              src/s2/util/gtl/densehashtable.h
              src/s2/util/gtl/hashtable_common.h
              src/s2/util/gtl/libc_allocator_with_realloc.h
              src/s2/util/gtl/manual_constructor.h
        DESTINATION include/s2/util/gtl)
install(FILES src/s2/util/hash/mix.h
        DESTINATION include/s2/util/hash)
install(FILES src/s2/util/math/mathutil.h
              src/s2/util/math/matrix3x3.h
              src/s2/util/math/vector.h
              src/s2/util/math/vector3_hash.h
        DESTINATION include/s2/util/math)
install(FILES src/s2/util/units/length-units.h
              src/s2/util/units/physical-units.h
        DESTINATION include/s2/util/units)
install(TARGETS s2 DESTINATION lib)

message("GTEST_ROOT: ${GTEST_ROOT}")
if (GTEST_ROOT)
  add_subdirectory(${GTEST_ROOT} build_gtest)

  set(S2TestFiles
      src/s2/id_set_lexicon_test.cc
      src/s2/mutable_s2shapeindex_test.cc
      src/s2/r1interval_test.cc
      src/s2/r2rect_test.cc
      src/s2/s1angle_test.cc
      src/s2/s1chordangle_test.cc
      src/s2/s1interval_test.cc
      src/s2/s2boolean_operation_test.cc
      src/s2/s2builder_graph_test.cc
      src/s2/s2builder_test.cc
      src/s2/s2builderutil_closed_set_normalizer_test.cc
      src/s2/s2builderutil_find_polygon_degeneracies_test.cc
      src/s2/s2builderutil_s2point_vector_layer_test.cc
      src/s2/s2builderutil_s2polygon_layer_test.cc
      src/s2/s2builderutil_s2polyline_layer_test.cc
      src/s2/s2builderutil_s2polyline_vector_layer_test.cc
      src/s2/s2builderutil_snap_functions_test.cc
      src/s2/s2builderutil_testing_test.cc
      src/s2/s2cap_test.cc
      src/s2/s2cell_test.cc
      src/s2/s2cellid_test.cc
      src/s2/s2cellunion_test.cc
      src/s2/s2centroids_test.cc
      src/s2/s2closestedgequery_base_test.cc
      src/s2/s2closestedgequery_test.cc
      src/s2/s2closestpointquery_base_test.cc
      src/s2/s2closestpointquery_test.cc
      src/s2/s2contains_point_query_test.cc
      src/s2/s2contains_vertex_query_test.cc
      src/s2/s2convexhullquery_test.cc
      src/s2/s2coords_test.cc
      src/s2/s2crossingedgequery_test.cc
      src/s2/s2debug_test.cc
      src/s2/s2earth_test.cc
      src/s2/s2edge_clipping_test.cc
      src/s2/s2edge_crosser_test.cc
      src/s2/s2edge_crossings_test.cc
      src/s2/s2edge_distances_test.cc
      src/s2/s2edge_tessellator_test.cc
      src/s2/s2edge_vector_shape_test.cc
      src/s2/s2error_test.cc
      src/s2/s2latlng_test.cc
      src/s2/s2latlngrect_bounder_test.cc
      src/s2/s2latlngrect_test.cc
      src/s2/s2lax_loop_shape_test.cc
      src/s2/s2lax_polygon_shape_test.cc
      src/s2/s2lax_polyline_shape_test.cc
      src/s2/s2loop_test.cc
      src/s2/s2measures_test.cc
      src/s2/s2metrics_test.cc
      src/s2/s2min_distance_targets_test.cc
      src/s2/s2paddedcell_test.cc
      src/s2/s2point_test.cc
      src/s2/s2point_vector_shape_test.cc
      src/s2/s2pointcompression_test.cc
      src/s2/s2pointindex_test.cc
      src/s2/s2pointregion_test.cc
      src/s2/s2pointutil_test.cc
      src/s2/s2polygon_test.cc
      src/s2/s2polyline_simplifier_test.cc
      src/s2/s2polyline_test.cc
      src/s2/s2predicates_test.cc
      src/s2/s2projections_test.cc
      src/s2/s2r2rect_test.cc
      src/s2/s2region_test.cc
      src/s2/s2region_term_indexer_test.cc
      src/s2/s2regioncoverer_test.cc
      src/s2/s2regionunion_test.cc
      src/s2/s2shapeindex_buffered_region_test.cc
      src/s2/s2shapeindex_region_test.cc
      src/s2/s2shapeindex_test.cc
      src/s2/s2shapeutil_build_polygon_boundaries_test.cc
      src/s2/s2shapeutil_contains_brute_force_test.cc
      src/s2/s2shapeutil_count_edges_test.cc
      src/s2/s2shapeutil_edge_iterator_test.cc
      src/s2/s2shapeutil_get_reference_point_test.cc
      src/s2/s2shapeutil_range_iterator_test.cc
      src/s2/s2shapeutil_test.cc
      src/s2/s2testing_test.cc
      src/s2/s2textformat_test.cc
      src/s2/s2wedge_relations_test.cc
      src/s2/sequence_lexicon_test.cc
      src/s2/value_lexicon_test.cc)

  enable_testing()

  foreach (test_cc ${S2TestFiles})
    get_filename_component(test ${test_cc} NAME_WE)
    add_executable(${test} ${test_cc})
    target_link_libraries(
        ${test}
        s2testing s2 gtest_main)
    add_test(${test} ${test})
  endforeach()
endif()

add_subdirectory("doc/examples" examples)

if (${SWIG_FOUND} AND ${PYTHONLIBS_FOUND})
  add_subdirectory("src/python" python)
endif()
