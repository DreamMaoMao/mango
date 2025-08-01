project('mango', ['c', 'cpp'],
  version : '0.8.0'
)

subdir('protocols')

is_nixos = false
os_release = run_command('cat', '/etc/os-release', check: false)
if os_release.returncode() == 0
    if os_release.stdout().contains('ID=nixos')
        is_nixos = true
    endif
endif

prefix = get_option('prefix')
sysconfdir = get_option('sysconfdir')

# 如果 sysconfdir 以 prefix 开头，去掉 prefix
if sysconfdir.startswith(prefix) and not is_nixos
  sysconfdir = sysconfdir.substring(prefix.length())
  # 确保 sysconfdir 是绝对路径
  if not sysconfdir.startswith('/')
    sysconfdir = '/' + sysconfdir
  endif
endif

# 打印调试信息，确认 sysconfdir 的值
# message('prefix: ' + prefix)
# message('sysconfdir: ' + sysconfdir)

cc = meson.get_compiler('c')
libm = cc.find_library('m')
xcb = dependency('xcb', required : get_option('xwayland'))
xlibs = dependency('xcb-icccm', required : get_option('xwayland'))
wayland_server_dep = dependency('wayland-server')
wlroots_dep = dependency('wlroots-0.19',version: '>=0.19.0')
xkbcommon_dep = dependency('xkbcommon')
libinput_dep = dependency('libinput')
libwayland_client_dep = dependency('wayland-client')
pcre2_dep = dependency('libpcre2-8')
libscenefx_dep = dependency('scenefx-0.4',version: '>=0.4.1')


# 获取 Git Commit Hash 和最新的 tag
git = find_program('git', required : false)
if git.found()
  commit_hash = run_command(git, 'rev-parse', '--short', 'HEAD', check : false).stdout().strip()
  latest_tag = run_command(git, 'describe', '--tags', '--abbrev=0', check : false).stdout().strip()
else
  commit_hash = 'unknown'
  latest_tag = meson.project_version()  # 如果 Git 不可用，使用默认版本号
endif

# 将 Commit Hash 和最新的 tag 添加到版本信息中
version_with_hash = '@0@(@1@)'.format(latest_tag, commit_hash)

# 定义编译参数
c_args = [
  '-g',
  '-Wno-unused-function',
  '-DWLR_USE_UNSTABLE',
  '-D_POSIX_C_SOURCE=200809L',
  '-DVERSION="@0@"'.format(version_with_hash),  # 版本信息包含 Commit Hash 和最新的 tag
  '-DSYSCONFDIR="@0@"'.format('/etc'),     # 添加 sysconfdir
]

if xcb.found() and xlibs.found()
  c_args += '-DXWAYLAND'
endif

executable('mango',
  'src/mango.c',
  'src/common/util.c',
  wayland_sources,
  dependencies : [
    libm,
    xcb,
    xlibs,
    libscenefx_dep,
    wayland_server_dep,
    wlroots_dep,
    xkbcommon_dep,
    libinput_dep,
    libwayland_client_dep,
    pcre2_dep,
  ],
  install : true,
  c_args : c_args
)

desktop_install_dir = join_paths(prefix, 'share/wayland-sessions')
install_data('mango.desktop', install_dir : desktop_install_dir)

# 安装 config.conf
install_data('config.conf', install_dir : join_paths(sysconfdir, 'mango'))
