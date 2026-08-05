#定义 App 源文件列表
# Application source files
set(USER_APP_SOURCES
    # App/App_Gimbal.cpp
    #Application/
)
#此处用Set创建了名为USER_APP_SOURCES的变量 它表示“所有 App 层源文件”。
# 以后真正创建文件后，例如创建了以下文件：
# Usercode/App/App_Gimbal.cpp
# Usercode/App/App_Communication.cpp

# 就将上面改成：
# set(USER_APP_SOURCES
#     App/App_Gimbal.cpp
#     App/App_Communication.cpp
# )
#是使用相对路径 也就是当前Usercode文件夹下的App文件夹
#例如上述实际指向：Usercode/App/App_Gimbal.cpp

#定义 Task 源文件列表
# FreeRTOS task source files
set(USER_TASK_SOURCES
    # Task/User_Task.cpp
    Gimbal_Task/Gimbal_Task.cpp
)
#使用方法同上

#合并以上App和Task两个列表

# ${变量名} 表示取出变量保存的内容。
# 假设前面是：
# set(USER_APP_SOURCES
#     App/App_Gimbal.cpp
# )

# set(USER_TASK_SOURCES
#     Task/User_Task.cpp
# )
# 那么执行合并后，${USERCODE_SOURCES} 相当于：
# App/App_Gimbal.cpp
# Task/User_Task.cpp

# Combine all Usercode source files 合并以上App和Task两个列表
set(USERCODE_SOURCES
    ${USER_APP_SOURCES}
    ${USER_TASK_SOURCES}
)
# 这里单独合并一次的好处是：
# App 和 Task 分类清楚。
# 后面只需要处理一个总列表。
# 以后增加 Bsp、Module 时可以继续扩展。
# 方便打印和排查哪些文件参与构建。
# 这些变量只在当前 CMake 目录和子目录中有效，不会污染整个系统环境。

#把源文件加入最终 ELF
# Add Usercode sources directly to the final ELF target
if(USERCODE_SOURCES)    #←检查总源文件列表是不是空的 若是空就不会执行下面的 target_sources() 语句（相当于没进if）
    target_sources(${CMAKE_PROJECT_NAME} PRIVATE #PRIVATE 表示这些源文件只属于当前 ELF 目标。
        ${USERCODE_SOURCES}
    )
    #↑把 ${USERCODE_SOURCES} 中的源文件直接追加到 ${CMAKE_PROJECT_NAME} 这个目标
    #在工程顶层目录的 CMakeLists.txt 中已经创建了这个目标。

    #最终逻辑
    #根 CMake
    # └─ add_executable() 创建最终 ELF
    #     └─ Usercode/CMakeLists.txt
    #         └─ target_sources() 把 App/Task 加入这个 ELF

endif()

#添加头文件搜索路径
# Make Usercode headers visible to this ELF target
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}"
)

# CMAKE_CURRENT_SOURCE_DIR
# 在 Usercode/CMakeLists.txt 中，它表示：
# D:\AAACE3.1\SteeringWheel_Infantry\SteeringWheel_Infantry_Gimbal\Usercode
# 因此编译器会把整个 Usercode 作为头文件搜索起点。
# 推荐在代码中这样包含头文件：
# #include "App/App_Gimbal.h"
# #include "Task/User_Task.h"
# 这样比单纯写：
# #include "App_Gimbal.h"
# 更清楚，也能避免以后 App 和 Task 出现同名头文件。

#CMake语法解释：
# set() 是什么？
# set() 用于创建或修改一个 CMake 变量：
# set(变量名
#     内容1
#     内容2
# )