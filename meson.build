# general project settings
project('stmh7', 'c', 'cpp',
  version : '1.0',
  default_options : [
    'b_lto=false',
    #'b_asneeded=false',
    #'b_lundef=false',
    'b_pch=false',
    'b_staticpic=false',
    'c_std=c99',
    'cpp_std=c++11'
    ],
  meson_version : '>=0.51.0')

# global variables tracking sources and includes
c_args = []
c_link_args = []
project_src = []
project_inc = []
test_src = []
test_inc = []
link_deps = []

# find external programs from path (OS independant)
size = find_program('arm-none-eabi-size')
objcopy = find_program('arm-none-eabi-objcopy')
stmprog = find_program(['STM32_Programmer_CLI',
  'C:\Program Files (x86)\STMicroelectronics\STM32Cube\STM32CubeProgrammer\\bin\STM32_Programmer_CLI.exe'],
 required : false)

# retrieve properties from cross compile file
if meson.is_cross_build()
  #c_args = meson.get_cross_property('c_cross_args')
  #c_link_args = meson.get_cross_property('c_cross_link_args')
endif

# microcontroller specific settings
defines = [
  '-DUSE_HAL_DRIVER',
  '-DSTM32H743xx',
]

linkerfile = files('src/STM32H743ZITx_FLASH.ld')
startupfile = files('src/startup_stm32h743xx.s')

# add defines to the c_args
c_args += defines
#c_args += '-U_FILE_OFFSET_BITS'

# add the linker script to flags
c_link_args += '-Wl,-T@0@/@1@'.format(meson.current_source_dir(), linkerfile[0])

# include doxygen generation if enabled
if get_option('enable_docs')
  subdir('docs')
endif

# include tests if enabled
if get_option('enable_test')
  subdir('test')
endif

# include the source files of the project
subdir('src')

# compile the main executable
main = executable('main',
  [project_src, startupfile],
  name_suffix : 'elf',
  c_args : c_args,
  link_args : [c_link_args, '-Wl,--gc-sections'],
  dependencies : link_deps,
  include_directories : [project_inc])

# cutom target to always convert main.elf to binary
mainbin = custom_target('main.bin',
  output : 'main.bin',
  build_by_default : true,
  command : [objcopy, ['-O', 'binary', '-S', main.full_path(), 'main.bin']], 
  depends : main)

# ouput size of the .elf
run_target('size',
  command : [size, main.full_path()],
  depends : main)

# upload the binary to the microcontroller
if not stmprog.found()
  warning('STM32_Programmer_CLI not found, flashing not possible')
else
  run_target('flash',
    command : [stmprog, '-c', 'port=SWD', '-w', mainbin.full_path(), '0x08000000', '-v', '-hardRst'],
    depends : mainbin)
endif
