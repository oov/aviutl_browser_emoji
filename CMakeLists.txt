cmake_minimum_required(VERSION 3.19)

project(aviutl_browser_emoji NONE)

find_package(Git REQUIRED)
execute_process(
  COMMAND ${GIT_EXECUTABLE} tag --points-at HEAD
  WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
  OUTPUT_VARIABLE git_tag
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE
)
if ("${git_tag}" STREQUAL "")
  set(git_tag "vX.X.X")
endif()

execute_process(
  COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
  WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
  OUTPUT_VARIABLE git_revision
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE
)
if ("${git_revision}" STREQUAL "")
  set(git_revision "unknown")
endif()

add_custom_target(generate_emoji_selector_html
  COMMAND ${CMAKE_COMMAND}
    -Dinput_file="${PROJECT_SOURCE_DIR}/src/emoji-selector/emoji-selector.html"
    -Doutput_file="${PROJECT_BINARY_DIR}/bin/script/browser/contents/絵文字セレクター/emoji-selector.html"
    -Dgit_tag="${git_tag}"
    -Dgit_revision="${git_revision}"
    -P "${PROJECT_SOURCE_DIR}/replace.cmake"
)
add_custom_target(create_emoji_selector
  COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/twemoji/assets/72x72" "${PROJECT_BINARY_DIR}/bin/script/browser/contents/絵文字セレクター/72x72"
)
add_dependencies(create_emoji_selector generate_emoji_selector_html)

add_custom_target(generate_content_html
  COMMAND ${CMAKE_COMMAND}
    -Dinput_file="${PROJECT_SOURCE_DIR}/src/content/index.html"
    -Doutput_file="${PROJECT_BINARY_DIR}/content/index.html"
    -Dgit_tag="${git_tag}"
    -Dgit_revision="${git_revision}"
    -P "${PROJECT_SOURCE_DIR}/replace.cmake"
)
add_custom_target(create_content
  COMMAND ${CMAKE_COMMAND} -E make_directory "${PROJECT_BINARY_DIR}/bin/script/browser/contents"
  COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/twemoji/assets/svg" "${PROJECT_BINARY_DIR}/content/svg"
  COMMAND ${CMAKE_COMMAND} -E tar cf "${PROJECT_BINARY_DIR}/bin/script/browser/contents/絵文字.abc" --format=zip "index.html" "svg" WORKING_DIRECTORY "${PROJECT_BINARY_DIR}/content"
)
add_dependencies(create_content generate_content_html)

add_custom_target(generate_script
  COMMAND ${CMAKE_COMMAND}
    -Dinput_file="${PROJECT_SOURCE_DIR}/src/emoji.obj"
    -Doutput_file="${PROJECT_BINARY_DIR}/bin/script/browser/絵文字.obj"
    -Dgit_tag="${git_tag}"
    -Dgit_revision="${git_revision}"
    -P "${PROJECT_SOURCE_DIR}/replace.cmake"
)

add_custom_target(make_package ALL
  COMMAND ${CMAKE_COMMAND}
    -Dinput_file="${PROJECT_SOURCE_DIR}/README.md"
    -Doutput_file="${PROJECT_BINARY_DIR}/bin/絵文字.txt"
    -Dgit_tag="${git_tag}"
    -Dgit_revision="${git_revision}"
    -P "${PROJECT_SOURCE_DIR}/replace.cmake"
)
add_dependencies(make_package
  create_emoji_selector
  create_content
  generate_script
)
