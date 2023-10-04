project('hello', 'c',
  default_options: [
    'warning_level=3',
    'b_ndebug=if-release',
    'b_pie=true',
    'b_lto=true'
  ]
)

executable('hello', 'main.c',
  dependencies: [
    dependency('glib-2.0')
  ]
)
