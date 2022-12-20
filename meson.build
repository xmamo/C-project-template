project('project', 'c',
  default_options: [
    'warning_level=3',
    'b_lto=true',
    'b_ndebug=if-release',
    'b_pie=true'
  ]
)

executable('project', 'source/main.c',
  depencencies: [
    dependency('glib-2.0')
  ]
)
