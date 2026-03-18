# Link Luau Compiler + Ast libraries
target_link_libraries(Remoter PRIVATE 
    kernel32 user32 winhttp ws2_32
    Luau.Compiler Luau.Ast
    zlibstatic
    libzstd_static
    blake3
    httplib::httplib
    nlohmann_json::nlohmann_json
)

# Add Luau include paths
target_include_directories(Remoter PRIVATE
    ${luau_SOURCE_DIR}/Compiler/include
    ${luau_SOURCE_DIR}/Ast/include
    ${luau_SOURCE_DIR}/Common/include
    ${zlib_SOURCE_DIR}
    ${zlib_BINARY_DIR}
    ${zstd_SOURCE_DIR}/lib
    ${zstd_SOURCE_DIR}/lib/common
    ${blake3_SOURCE_DIR}/c
)
