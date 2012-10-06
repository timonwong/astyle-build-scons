import platform


ASTYLE_SOURCE_DIR = './astyle/src/'

# Options
options = {}
options_topass = {}


def add_option(name, help, nargs, dest=None, default=None,
               type="string", choices=None):
    if dest is None:
        dest = name

    AddOption("--" + name,
              dest=dest,
              type=type,
              nargs=nargs,
              action="store",
              choices=choices,
              default=default,
              help=help)

    options[name] = {"help": help,
                     "nargs": nargs,
                     "dest": dest,
                     "default": default}


def get_option(name):
    return GetOption(name)


def _has_option(name):
    x = get_option(name)
    if x is None:
        return False
    if x == False:
        return False
    if x == "":
        return False
    return True


def has_option(name):
    x = _has_option(name)
    if name not in options_topass:
        # if someone already set this, don't overwrite
        options_topass[name] = x
    return x

# base compile flags
add_option("64", "Whether to force 64 bit (No effect on OSX)", 0, "force64")
add_option("32", "Whether to force 32 bit (No effect on OSX)", 0, "force32")


# setup environment
force32 = has_option("force32")
force64 = has_option("force64")

target_arch = platform.machine().lower()
if force32:
    target_arch = 'x86'
if force64:
    target_arch = 'x86_64'

env = Environment(
    CPPDEFINES=['ASTYLE_LIB'],
    CPPPATH=ASTYLE_SOURCE_DIR,
    TARGET_ARCH = target_arch
)

plat = env['PLATFORM']
arch = env['TARGET_ARCH']

if arch == 'x86_64' or arch == 'amd64' or arch == 'emt64':
    arch = 'x86_64'
elif arch == 'x86' or arch == 'i386':
    arch = 'x86'
else:
    # not supported
    raise Exception("Unsupported architecture: %s" % arch)

# Files those need to be compiled and linked with
cppfiles = """
    ASBeautifier.cpp
    ASEnhancer.cpp
    ASFormatter.cpp
    ASResource.cpp
    astyle_main.cpp
""".split()

variant_sub_dir = "%s_%s" % (plat, arch)
lib_filename = 'astyle'

if plat == 'win32':  # Windows
    if arch == 'x86_64' or arch == 'amd64' or arch == 'emt64':
        lib_filename = 'AStyle_x64'
    else:
        lib_filename = 'AStyle'

    env.PrependUnique(CPPDEFINES='NDEBUG')
    env.AppendUnique(CXXFLAGS=['/W3', '/O2', '/GR-', '/EHsc', '/MT'])
    env.AppendUnique(LINKFLAGS=['/MANIFEST'])
elif plat == 'darwin':  # Darwin
    variant_sub_dir = plat

    env.AppendUnique(CXXFLAGS=['-arch i386', '-arch x86_64'])
    env.AppendUnique(LINKFLAGS=['-arch i386', '-arch x86_64'])
else:  # Linux 'POSIX'
    if arch == 'x86_64' or arch == 'amd64' or arch == 'emt64':
        lib_filename = 'astyle64'
        env.AppendUnique(CXXFLAGS='-m64')
        env.AppendUnique(LINKFLAGS='-m64')
    else:
        env.AppendUnique(CXXFLAGS='-m32')
        env.AppendUnique(LINKFLAGS='-m32')

if plat != 'win32':
    env.PrependUnique(CXXFLAGS=['-Wall', '-O2'])

variant_dir = './build/%s/' % (variant_sub_dir)
env.VariantDir(variant_dir, ASTYLE_SOURCE_DIR, duplicate=0)

build_result = env.SharedLibrary(target=variant_dir + lib_filename,
                                 source=[variant_dir + fn for fn in cppfiles])

if plat == 'win32':
    # Embeding manifest
    env.AddPostAction(build_result, 'mt.exe -nologo -manifest ${TARGET}.manifest -outputresource:$TARGET;2')
