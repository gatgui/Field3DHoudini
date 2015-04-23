import excons
from excons.tools import houdini

import os, re, sys, glob, subprocess, shutil

# Copy sources from houdini HDK
src = "f3dtools.C"
if os.path.isfile(src):
  os.remove(src)

hver, hfs = houdini.GetVersionAndDirectory(noexc=True)
if hfs:
  srcdir = hfs + "/toolkit/samples/field3d"
else:
  srcdir = None

if not srcdir or not os.path.isdir(srcdir):
  print("Could not figure out Houdini field3d DSO sources location")
  sys.exit(1)

for f in glob.glob(srcdir+"/*.C"):
  shutil.copyfile(f, os.path.basename(f))

for f in glob.glob(srcdir+"/*.h"):
  shutil.copyfile(f, os.path.basename(f))

if not os.path.isfile(src):
  print("Source file \"%s\" doesn't exist" % src)
  sys.exit(1)

# Field3D setup
Field3D_build = False
Field3D_static = False

Field3D_inc, Field3D_lib = excons.GetDirs("field3d", noexc=True)

if not Field3D_inc and not Field3D_lib:
  f3d = excons.GetArgument("with-field3d", None)
  
  if not f3d or f3d == "houdini":
    # Use version provided with houdini
    if sys.platform == "win32":
      Field3D_inc = "%s/toolkit/include" % hfs
      Field3D_lib = "%s/custom/houdini/dsolib" % hfs
    
    elif sys.platform == "darwin":
      Field3D_inc = "%s/Resources/toolkit/include" % hfs
      Field3D_lib = "%s/Libraries" % hfs
    
    else:
      Field3D_inc = "%s/toolkit/include" % hfs
      Field3D_lib = "%s/dsolib" % hfs
  
  else:
    # Build our own (don't forget then to set field3d build flags for boost/ilmbase/hdf5)
    Field3D_build = True
    Field3D_prefix = os.path.abspath("./Field3D/%s/%s" % (excons.mode_dir, excons.arch_dir))
    Field3D_inc = "%s/include" % Field3D_prefix
    Field3D_lib = "%s/lib" % Field3D_prefix

# Build Field3D if needed
if Field3D_build:
  Field3D_static = (excons.GetArgument("field3d-static", 0, int) != 0)
  excons.SetArgument("static", "1" if Field3D_static else "0")
  SConscript("Field3D/SConstruct")

# Build houdini DSO
incdirs = [Field3D_inc]
libdirs = [Field3D_lib]
libs = ["Field3D"]
defs = []
if Field3D_static:
  defs.append("FIELD3D_STATIC")

targets = [
  {"name"    : "houdini%s/dso/f3dtools" % hver,
   "alias"   : "f3dtools",
   "type"    : "dynamicmodule",
   "ext"     : houdini.PluginExt(),
   "defs"    : defs,
   "srcs"    : [src],
   "incdirs" : incdirs,
   "libdirs" : libdirs,
   "libs"    : libs,
   "custom"  : [houdini.Require, houdini.Plugin]}
]

env = excons.MakeBaseEnv()
excons.DeclareTargets(env, targets)

Default(["f3dtools"])
