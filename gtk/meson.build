project('snes9x-gtk',
        ['c', 'cpp'],
        version: '1.61',
        meson_version: '>=0.56.2',
        subproject_dir: 'external',
        default_options: ['cpp_std=c++17'])

args = ['-DSNES9X_GTK', '-DUNZIP_SUPPORT', '-DNETPLAY_SUPPORT', '-DJMA_SUPPORT', '-Wall', '-W', '-Wno-unused-parameter']
srcs = []
deps = []
includes = ['../apu/bapu', '../', 'src', '../emu-nwaccess', '../emu-nwaccess/emulator-networkaccess/c_includes']
warns = []
libs = []

prefix = get_option('prefix')
localedir = join_paths(prefix, get_option('localedir'))
datadir = get_option('datadir')
appdatadir = get_option ('appdatadir')
if appdatadir == ''
  appdatadir = join_paths(prefix, datadir, 'snes9x')
else
  appdatadir = join_paths(prefix, datadir, appdatadir)
endif
args += [ '-DDATADIR="' + appdatadir + '"', '-DSNES9XLOCALEDIR="' + localedir + '"' ]

subdir('data')
subdir('po')

sdl2_dep = dependency('sdl2')

deps += sdl2_dep

c_compiler = meson.get_compiler('c')

if c_compiler.version().version_compare('>=7.0.0') and c_compiler.get_id() == 'gcc'
  args += '-Wno-format-truncation'
endif

gtk_dep = dependency('gtkmm-3.0', version: '>= 3.22')
deps += gtk_dep

gthread_dep = dependency('gthread-2.0')
deps += gthread_dep

x11_dep = c_compiler.find_library('X11')
xext_dep = c_compiler.find_library('Xext')
dl_dep = c_compiler.find_library('dl')
deps += x11_dep
deps += xext_dep
deps += dl_dep

xrandr_dep = dependency('xrandr')
deps += xrandr_dep

opengl_dep = dependency('epoxy', required: true)
srcs += [ 'src/gtk_display_driver_opengl.cpp',
          'src/gtk_display_driver_opengl.h',
          'src/gtk_glx_context.cpp',
          'src/gtk_glx_context.h',
          '../shaders/glsl.cpp',
          '../shaders/shader_helpers.cpp',
          'src/gtk_shader_parameters.cpp' ]
deps += opengl_dep

slang = get_option('slang')
if slang

  cmake = import('cmake')

  glslang_opt_var = cmake.subproject_options()
  glslang_opt_var.add_cmake_defines({'BUILD_TESTING': false})
  glslang_pro = cmake.subproject('glslang', options: glslang_opt_var)
#  deps += glslang_pro.dependency('GenericCodeGen')
  deps += glslang_pro.dependency('glslang')
#  deps += glslang_pro.dependency('MachineIndependent')
  deps += glslang_pro.dependency('OGLCompiler')
  deps += glslang_pro.dependency('HLSL')
  deps += glslang_pro.dependency('OSDependent')
  deps += glslang_pro.dependency('SPIRV')
  deps += glslang_pro.dependency('glslang-default-resource-limits')

  spirv_cross_opt_var = cmake.subproject_options()
  spirv_cross_opt_var.add_cmake_defines({'SPIRV_CROSS_EXCEPTIONS_TO_ASSERTIONS': true})
  spirv_cross_pro = cmake.subproject('SPIRV-Cross', options: spirv_cross_opt_var)
  deps += spirv_cross_pro.dependency('spirv-cross-core')
  deps += spirv_cross_pro.dependency('spirv-cross-glsl')
  deps += spirv_cross_pro.dependency('spirv-cross-reflect')
  deps += spirv_cross_pro.dependency('spirv-cross-cpp')
  
  args += ['-DUSE_SLANG']
  srcs += '../shaders/slang.cpp'
  includes += '../external/glslang'
endif

wayland = get_option('wayland')
if wayland
  wayland_client_dep = dependency('wayland-client', required: false)
  wayland_dep = dependency('wayland-egl', required: false)

  if wayland_dep.found() and wayland_client_dep.found()
    args += '-DUSE_WAYLAND'
    srcs += ['src/gtk_wayland_egl_context.cpp', 'src/gtk_wayland_egl_context.h', 'src/wayland-idle-inhibit-unstable-v1.c']
    deps += [wayland_dep, wayland_client_dep]
  else
    wayland = false
    warns += 'wayland-egl not found. Wayland will be disabled.'
  endif
endif

xv = get_option('xv')
if xv
  xv_dep = dependency('xv', required: false)

  if xv_dep.found()
    args += '-DUSE_XV'
    srcs += ['src/gtk_display_driver_xv.cpp', 'src/gtk_display_driver_xv.h']
    deps += xv_dep
  else
    xv = false
    warns += 'XV/XVideo was not found. It will be disabled.'
  endif
endif

portaudio = get_option('portaudio')
if portaudio
  portaudio_dep = dependency('portaudio-2.0', version: '>= 10', required: false)

  if portaudio_dep.found()
    args += '-DUSE_PORTAUDIO'
    srcs += ['src/gtk_sound_driver_portaudio.cpp', 'src/gtk_sound_driver_portaudio.h']
    deps += portaudio_dep
  else
    portaudio = false
    warns += 'PortAudio was not found. It will disabled.'
  endif
endif

oss = get_option('oss')
if oss
  if c_compiler.has_header('sys/soundcard.h')
    args += '-DUSE_OSS'
    srcs += ['src/gtk_sound_driver_oss.cpp', 'src/gtk_sound_driver_oss.h']
  else
    oss = false
    warns += 'OSS not found. It will be disabled.'
  endif
endif

alsa = get_option('alsa')
if alsa
  alsa_dep = dependency('alsa', required: false)

  if alsa_dep.found()
    args += '-DUSE_ALSA'
    srcs += ['src/gtk_sound_driver_alsa.cpp', 'src/gtk_sound_driver_alsa.h']
    deps += alsa_dep
  else
    alsa = false
    warns += 'ALSA not found. It will be disabled.'
  endif
endif

pulseaudio = get_option('pulseaudio')
if pulseaudio
  pulseaudio_dep = dependency('libpulse', required: false)

  if pulseaudio_dep.found()
    args += '-DUSE_PULSEAUDIO'
    srcs += ['src/gtk_sound_driver_pulse.cpp', 'src/gtk_sound_driver_pulse.h']
    deps += pulseaudio_dep
  else
    pulseaudio = false
    warns += 'PulseAudio not found. Disabling.'
  endif
endif

screenshot = get_option('screenshot')
if screenshot
  screenshot_dep = dependency('libpng', required: false)

  if screenshot_dep.found()
    args += '-DHAVE_LIBPNG'
    deps += screenshot_dep
  else
    screenshot = false
    warns += 'libpng not found. Disabling screenshot support.'
  endif
endif

systemzip_dep = dependency('minizip', required: false)
systemzip = get_option('system-zip')
if systemzip and systemzip_dep.found()
  args += '-DSYSTEM_ZIP'
  deps += systemzip_dep
else
  systemzip = false
  includes += '../unzip'
  srcs += ['../unzip/unzip.c', '../unzip/ioapi.c', '../unzip/zip.c']
endif

zlib = get_option('zlib')
if zlib
  zlib_dep = dependency('zlib', required: false)

  if zlib_dep.found()
    args += '-DZLIB'
    deps += zlib_dep
  else
    zlib = false
    warns += 'zlib not found. Disabling.'
  endif
endif

if get_option('dangerous-hacks')
  warns += 'Dangerous hacks are enabled. Don\'t complain if things break!'
  args += '-DALLOW_CPU_OVERCLOCK'
endif

if get_option('hq2x')
  args += '-DUSE_HQ2X'
  srcs += ['../filter/hq2x.cpp', '../filter/hq2x.h']
endif

if get_option('xbrz')
  args += '-DUSE_XBRZ'
  srcs += ['../filter/xbrz.cpp', '../filter/xbrz.h', 'src/filter_xbrz.cpp', 'src/filter_xbrz.h']
endif

args += ['-DHAVE_MKSTEMP', '-DHAVE_STRINGS_H', '-DHAVE_STDINT_H', '-DRIGHTSHIFT_IS_SAR']

srcs += [
 'src/gtk_sound_driver.h',
 '../filter/2xsai.cpp',
 '../filter/2xsai.h',
 '../filter/epx.cpp',
 '../filter/epx.h',
 'src/filter_epx_unsafe.h',
 'src/filter_epx_unsafe.cpp',
 'src/gtk_binding.cpp',
 'src/gtk_binding.h',
 'src/gtk_cheat.cpp',
 'src/gtk_cheat.h',
 'src/gtk_config.cpp',
 'src/gtk_config.h',
 'src/gtk_control.cpp',
 'src/gtk_control.h',
 'src/gtk_display.cpp',
 'src/gtk_display_driver_gtk.cpp',
 'src/gtk_display_driver_gtk.h',
 'src/gtk_display_driver.h',
 'src/gtk_display.h',
 'src/gtk_file.cpp',
 'src/gtk_file.h',
 'src/gtk_builder_window.cpp',
 'src/gtk_builder_window.h',
 'src/gtk_preferences.cpp',
 'src/gtk_preferences.h',
 'src/gtk_s9xcore.h',
 'src/gtk_s9x.cpp',
 'src/gtk_s9x.h',
 'src/gtk_s9xwindow.cpp',
 'src/gtk_s9xwindow.h',
 'src/gtk_sound.cpp',
 'src/gtk_sound.h',
 'src/gtk_splash.cpp',
 '../filter/snes_ntsc_config.h',
 '../filter/snes_ntsc.h',
 '../filter/snes_ntsc_impl.h',
 '../filter/snes_ntsc.c',
 'src/gtk_compat.h',
 'src/gtk_sound_driver_sdl.h',
 'src/gtk_sound_driver_sdl.cpp',
 '../fxinst.cpp',
 '../fxemu.cpp',
 '../fxdbg.cpp',
 '../c4.cpp',
 '../c4emu.cpp',
 '../apu/apu.cpp',
 '../apu/bapu/dsp/sdsp.cpp',
 '../apu/bapu/smp/smp.cpp',
 '../apu/bapu/smp/smp_state.cpp',
 '../msu1.cpp',
 '../msu1.h',
 '../dsp.cpp',
 '../dsp1.cpp',
 '../dsp2.cpp',
 '../dsp3.cpp',
 '../dsp4.cpp',
 '../spc7110.cpp',
 '../obc1.cpp',
 '../seta.cpp',
 '../seta010.cpp',
 '../seta011.cpp',
 '../seta018.cpp',
 '../controls.cpp',
 '../crosshairs.cpp',
 '../cpu.cpp',
 '../sa1.cpp',
 '../debug.cpp',
 '../sdd1.cpp',
 '../tile.cpp',
 '../tileimpl-n1x1.cpp',
 '../tileimpl-n2x1.cpp',
 '../tileimpl-h2x1.cpp',
 '../srtc.cpp',
 '../gfx.cpp',
 '../memmap.cpp',
 '../clip.cpp',
 '../ppu.cpp',
 '../dma.cpp',
 '../snes9x.cpp',
 '../globals.cpp',
 '../stream.cpp',
 '../conffile.cpp',
 '../bsx.cpp',
 '../snapshot.cpp',
 '../screenshot.cpp',
 '../movie.cpp',
 '../statemanager.cpp',
 '../sha256.cpp',
 '../bml.cpp',
 '../cpuops.cpp',
 '../cpuexec.cpp',
 '../sa1cpu.cpp',
 '../cheats.cpp',
 '../cheats2.cpp',
 '../sdd1emu.cpp',
 '../netplay.cpp',
 '../server.cpp',
 '../loadzip.cpp',
 '../compat.cpp',
 'src/gtk_netplay_dialog.cpp',
 'src/gtk_netplay_dialog.h',
 'src/gtk_netplay.cpp',
 'src/gtk_netplay.h',
 'src/background_particles.cpp',
 'src/background_particles.h',
 '../emu-nwaccess/snes9x-nwaccess.cpp',
 '../emu-nwaccess/snes9x-nwaccess.h'
]

libjma_srcs = [
 '../jma/s9x-jma.cpp',
 '../jma/7zlzma.cpp',
 '../jma/crc32.cpp',
 '../jma/iiostrm.cpp',
 '../jma/inbyte.cpp',
 '../jma/jma.cpp',
 '../jma/lzma.cpp',
 '../jma/lzmadec.cpp',
 '../jma/winout.cpp'
]

libjma = static_library('jma',
                        libjma_srcs,
                        c_args: args,
                        cpp_args: args,
                        include_directories: include_directories(includes))
libs += libjma

sourcify = executable('sourcify', 'src/sourcify.c', native: true)
gtk_snes9x_ui_cpp = custom_target('sourcify',
                                  input: 'src/snes9x.ui',
                                  output: 'gtk_snes9x_ui.cpp',
                                  command: [sourcify, '@INPUT@', '@OUTPUT@', 'snes9x_ui'])

mini_icon = custom_target('mini_icon',
                          input: 'data/mini_icon.png',
                          output: 'mini_icon.cpp',
                          command: [sourcify, '@INPUT@', '@OUTPUT@', 'mini_icon'])

snes9x_gtk = executable('snes9x-gtk',
                        srcs,
                        gtk_snes9x_ui_cpp,
                        mini_icon,
                        c_args: args,
                        cpp_args: args,
                        dependencies: deps,
                        include_directories: include_directories(includes),
                        link_with: libs,
                        install: true)

summary = [
  'Snes9x GTK+ Build Configuration',
  '[Locations] prefix:            ' + prefix,
  '            datadir:           ' + datadir,
  '            appdatadir:        ' + appdatadir,
  '            localedir:         ' + localedir,
  '[Options]   Build type:        ' + get_option('buildtype'),
  '            Wayland:           ' + wayland.to_string(),
  '            slang shaders:     ' + slang.to_string(),
  '            XVideo:            ' + xv.to_string(),
  '            ALSA:              ' + alsa.to_string(),
  '            Open Sound System: ' + oss.to_string(),
  '            PulseAudio:        ' + pulseaudio.to_string(),
  '            PortAudio:         ' + portaudio.to_string(),
  '            HQ2X filter:       ' + get_option('hq2x').to_string(),
  '            xBRZ filter:       ' + get_option('xbrz').to_string(),
  '            Screenshot saving: ' + screenshot.to_string(),
  '            zlib compression:  ' + zlib.to_string(),
  '            System minizip:    ' + systemzip.to_string(),
]

summary += warns
message('\n'.join(summary))
