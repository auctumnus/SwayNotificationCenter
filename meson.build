project('sway-notificaton-center', ['c', 'vala'],
  version: '0.7.3',
  meson_version: '>= 0.50.0',
  default_options: [ 'warning_level=2' ],
)

add_project_arguments(['--enable-gobject-tracing'], language: 'vala')
add_project_arguments(['--enable-checking'], language: 'vala')

i18n = import('i18n')

subdir('data')
subdir('src')

datadir = get_option('datadir')
libdir = get_option('libdir')

conf_data = configuration_data()
conf_data.set('bindir', join_paths(get_option('prefix'), get_option('bindir')))

# Dbus service
configure_file(
  configuration: conf_data,
  input: 'services/dbus/org.erikreider.swaync.service.in',
  output: '@BASENAME@',
  install_dir: datadir + '/dbus-1/services'
)

# Systemd service unit
systemd = dependency('systemd', required: false)
if get_option('systemd-service')
  if systemd.found()
    systemd_service_install_dir = systemd.get_pkgconfig_variable('systemduserunitdir')
  else
    systemd_service_install_dir = join_paths(libdir, 'systemd', 'user')
  endif

  configure_file(
    configuration: conf_data,
    input: 'services/systemd/swaync.service.in',
    output: '@BASENAME@',
    install_dir: systemd_service_install_dir
  )
endif

# Zsh completion
if get_option('zsh-completions')
  zsh_files = files(
    'completions/zsh/_swaync',
    'completions/zsh/_swaync-client',
  )
  zsh_install_dir = join_paths(datadir, 'zsh', 'site-functions')

  install_data(zsh_files, install_dir: zsh_install_dir)
endif

# Bash completion
bash_comp = dependency('bash-completion', required: false)
if get_option('bash-completions')
  bash_files = files(
    'completions/bash/swaync',
    'completions/bash/swaync-client',
  )

  if bash_comp.found()
    bash_install_dir = bash_comp.get_variable(
      pkgconfig: 'completionsdir',
      pkgconfig_define: ['datadir', datadir]
    )
  else
    bash_install_dir = join_paths(datadir, 'bash-completion', 'completions')
  endif

  install_data(bash_files, install_dir: bash_install_dir)
endif

# Fish completion
fish_comp = dependency('fish', required: false)
if get_option('fish-completions')
  fish_files = files(
    'completions/fish/swaync.fish',
    'completions/fish/swaync-client.fish',
  )

  if fish_comp.found()
    fish_install_dir = fish_comp.get_variable(
      pkgconfig: 'completionsdir',
      pkgconfig_define: ['datadir', datadir]
    )
  else
    fish_install_dir = join_paths(datadir, 'fish', 'vendor_completions.d')
  endif

  install_data(fish_files, install_dir: fish_install_dir)
endif

# Man pages
if get_option('man-pages')
  scdoc = dependency('scdoc', version: '>=1.9.2', native: true, required: true)
  if scdoc.found()
    scdoc_prog = find_program(scdoc.get_pkgconfig_variable('scdoc'), native: true)

    mandir = get_option('mandir')
    man_files = [
      'swaync.1.scd',
      'swaync.5.scd',
      'swaync-client.1.scd',
    ]

    foreach filename : man_files
      topic = filename.split('.')[-3].split('/')[-1]
      section = filename.split('.')[-2]
      output = '@0@.@1@'.format(topic, section)
      message(mandir, section, '@0@/man@1@'.format(mandir, section))

      custom_target(
        output,
        input: join_paths('man', filename),
        output: output,
        command: scdoc_prog,
        install: true,
        feed: true,
        capture: true,
        install_dir: '@0@/man@1@'.format(mandir, section)
      )
    endforeach
  endif
endif

# Run the postinstall script when installing
meson.add_install_script('build-aux/meson/postinstall.py')
