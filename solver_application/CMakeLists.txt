project(solver_app)
add_executable(solver_app equation.cpp)
target_link_libraries(solver_app PRIVATE formatter_ex solver)
install(TARGETS solver_app DESTINATION bin)
