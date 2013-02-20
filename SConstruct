import excons
from excons.tools import houdini

import os, re, sys, glob, subprocess, shutil

houver = ARGUMENTS.get("houdini-ver", None)
if not houver:
  print("=== Using Houdini 12.1.185, override using houdini-ver= or with-houdini=")
  houver = "12.1.185"
  ARGUMENTS["houdini-ver"] = houver

# Copy sources from houdini HDK
src = "f3dtools.C"
if os.path.isfile(src):
  os.remove(src)
_, hfs = houdini.GetVersionAndDirectory(noexc=True)
srcdir = None
if hfs:
  if sys.platform == "darwin":
    srcdir = hfs + "/Resources/toolkit/samples/field3d"
  else:
    srcdir = hfs + "/toolkit/samples/field3d"
if not srcdir or not os.path.isdir(srcdir):
  raise Exception("Could not figure out Houdini field3d DSO sources location")
for f in glob.glob(srcdir+"/*.C"):
  shutil.copy2(f, os.path.basename(f))
for f in glob.glob(srcdir+"/*.h"):
  shutil.copy2(f, os.path.basename(f))
if not os.path.isfile(src):
  raise Exception("Source file \"%s\" doesn't exist" % src)
else:
  if sys.platform != "win32":
    # There's a little issue with sources in Houdini 12.0 (fixed in 12.1):
    # It defines OPENEXR_DLL on non-windows platform, which leads to a compile error
    oede = re.compile(r"#define\s+OPENEXR_DLL")
    f0 = open(src+".tmp", "w")
    f1 = open(src, "r")
    for l in f1.readlines():
      if oede.search(l):
        f0.write("//"+l)
      else:
        f0.write(l)
    f0.close()
    f1.close()
    os.remove(src)
    shutil.move(src+".tmp", src)

# Field3D setup
if not ARGUMENTS.get("with-field3d", None):
  Field3D_build = True
  Field3D_prefix = os.path.abspath("./Field3D/install/%s/%s/release" % (sys.platform, "m64" if excons.Build64() else "m32"))
  Field3D_inc = "%s/include" % Field3D_prefix
  Field3D_lib = "%s/lib" % Field3D_prefix
  ARGUMENTS["with-field3d-inc"] = Field3D_inc
  ARGUMENTS["with-field3d-lib"] = Field3D_lib
else:
  Field3D_inc, Field3D_lib = excons.GetDirs("field3d")

# OpenMPI setup
OpenMPI_inc, OpenMPI_lib = None, None
try:
  s = houver.split(".")
  maj = int(s[0])
  min = int(s[1])
except:
  raise Exception("Invalid houdini version \"%s\"" % houver)
if maj == 12 and min == 0:
  OpenMPI_def_prefix = None
  OpenMPI_def_inc = None
  OpenMPI_def_lib = None
  if sys.platform == "darwin":
    OpenMPI_def_prefix = "/opt/local"
  elif sys.platform == "linux2":
    OpenMPI_def_inc = "/usr/include/openmpi"
    if excons.Build64():
      OpenMPI_def_inc += "-x86_64"
      OpenMPI_def_lib = "/usr/lib64"
    else:
      OpenMPI_def_lib = "/usr/lib"
  OpenMPI_inc, OpenMPI_lib = excons.GetDirs("openmpi", OpenMPI_def_prefix, OpenMPI_def_inc, OpenMPI_def_lib)

# Build Field3D if needed
if Field3D_build:
  Field3D_prefix = "./Field3D/install/%s/%s/release" % (sys.platform, "m64" if excons.arch_dir == "x64" else "m32")
  env = Environment()
  houdini.Require(env)
  if OpenMPI_inc:
    env.Append(CPPPATH = [OpenMPI_inc])
  if OpenMPI_lib:
    env.Append(LIBPATH = [OpenMPI_lib])
  Export("env")
  SConscript("Field3D/SConscript")

# Build houdini DSO
incdirs = [Field3D_inc]
if OpenMPI_inc:
  incdirs.append(OpenMPI_inc)
libdirs = [Field3D_lib]
if OpenMPI_lib:
  libdirs.append(OpenMPI_lib)

targets = [
  {"name"    : "houdini%s/dso/f3dtools" % ARGUMENTS.get("houdini-ver"),
   "alias"   : "f3dtools",
   "type"    : "dynamicmodule",
   "ext"     : houdini.PluginExt(),
   "srcs"    : [src],
   "incdirs" : incdirs,
   "libdirs" : libdirs,
   "libs"    : ["Field3D"],
   "custom"  : [houdini.Require, houdini.Plugin]}
]

env = excons.MakeBaseEnv()
excons.DeclareTargets(env, targets)

Default(["f3dtools"])
