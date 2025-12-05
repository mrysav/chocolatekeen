#---------------------------------------------------------------------------------
ifeq ($(strip $(DEVKITPPC)),)
$(error "Please set DEVKITPPC in your environment. export DEVKITPPC=<path to>devkitPPC")
endif

include $(DEVKITPRO)/libogc2/gamecube_rules
export SDLCONFIG	:=	`$(DEVKITPRO)/portlibs/gamecube/bin/sdl2-config`
export SDL_CFLAGS 	:=	`$(SDLCONFIG) --cflags`
export SDL_LIBS		:=	`$(SDLCONFIG) --libs`
export SDL_INCLUDE	:=	"$(DEVKITPRO)/portlibs/gamecube/include/SDL2"

#---------------------------------------------------------------------------------
# TARGET is the name of the output
# BUILD is the directory where object files & intermediate files will be placed
# SOURCES is a list of directories containing source code
# INCLUDES is a list of directories containing extra header files
#---------------------------------------------------------------------------------
TARGET		:=	chocolatekeen-gc
TARGETDIR   :=  executables
BUILD		:=	build_gc
SOURCES		:=	../../src ../../src/decompression \
                ../../src/decompression/CGenius \
				../../src/decompression/CGenius/compression \
				../../src/decompression/CGenius/compression/fileio
DATA		:=	../../rsrc
INCLUDES	:=	../../src

#---------------------------------------------------------------------------------
# options for code generation
#---------------------------------------------------------------------------------
SHAREDFLAGS	=	-g -O2 -Wall $(MACHDEP) $(INCLUDE) $(SDL_CFLAGS) \
				-DFRAMESKIP -DPSS_STYLE=1 -DPATH_MAX=1024 -DHAVE_ASPRINTF \
				-D_SZ_ONE_DIRECTORY -D_LZMA_IN_CB -D_LZMA_OUT_READ -DNOUNCRYPT -DNO_SOUND -DUSE_VM \
				-fomit-frame-pointer \
				-Wno-format -Wno-format-overflow -Wno-stringop-overflow -Wno-format-truncation -Wno-unused-label \
				-Wno-unused-parameter -Wno-strict-aliasing -Wno-write-strings -Wno-stringop-truncation -Wno-sign-compare \
				-Wno-unused-variable -Wno-parentheses -Wno-unused-function -Wno-switch -Wno-unused-but-set-variable \
				-Wno-narrowing -Wno-misleading-indentation -Wno-restrict -Wno-maybe-uninitialized -Wno-infinite-recursion
CFLAGS		=	$(SHAREDFLAGS) -Wno-incompatible-pointer-types
CXXFLAGS	=	$(SHAREDFLAGS) -std=c99 -Wno-catch-value -Wno-class-memaccess
LDFLAGS		=	-g $(MACHDEP) -Wl,-Map,$(notdir $@).map
LDFLAGS		+=	-L../buildtools

#---------------------------------------------------------------------------------
# any extra libraries we wish to link with the project
#---------------------------------------------------------------------------------
LIBS	:= -lpng -lmxml -ltinysmb -lbba -lfat -liso9660 -lasnd -lz -logc $(SDL_LIBS)

#---------------------------------------------------------------------------------
# list of directories containing libraries, this must be the top level containing
# include and lib
#---------------------------------------------------------------------------------
LIBDIRS	:= $(PORTLIBS)


#---------------------------------------------------------------------------------
# no real need to edit anything past this point unless you need to add additional
# rules for different file extensions
#---------------------------------------------------------------------------------
ifneq ($(BUILD),$(notdir $(CURDIR)))
#---------------------------------------------------------------------------------

export OUTPUT	:=	$(CURDIR)/$(TARGETDIR)/$(TARGET)

export VPATH	:=	$(foreach dir,$(SOURCES),$(CURDIR)/$(dir)) \
					$(foreach dir,$(DATA),$(CURDIR)/$(dir))

export DEPSDIR	:=	$(CURDIR)/$(BUILD)

#---------------------------------------------------------------------------------
# automatically build a list of object files for our project
#---------------------------------------------------------------------------------
CFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.c)))
CPPFILES	:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.cpp)))
sFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.s)))
SFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.S)))
BINFILES	:=	$(foreach dir,$(DATA),$(notdir $(wildcard $(dir)/*.ttf) \
					$(wildcard $(dir)/*.lang) $(wildcard $(dir)/*.png) \
					$(wildcard $(dir)/*.pcm)))

#---------------------------------------------------------------------------------
# use CXX for linking C++ projects, CC for standard C
#---------------------------------------------------------------------------------
ifeq ($(strip $(CPPFILES)),)
	export LD	:=	$(CC)
else
	export LD	:=	$(CXX)
endif

export OFILES_BIN	:=	$(addsuffix .o,$(BINFILES))
export OFILES_SOURCES := $(CPPFILES:.cpp=.o) $(CFILES:.c=.o) $(sFILES:.s=.o) $(SFILES:.S=.o)
export OFILES := $(OFILES_BIN) $(OFILES_SOURCES)

export HFILES := $(addsuffix .h,$(subst .,_,$(BINFILES)))

#---------------------------------------------------------------------------------
# build a list of include paths
#---------------------------------------------------------------------------------
export INCLUDE	:=	$(foreach dir,$(INCLUDES), -iquote $(CURDIR)/$(dir)) \
					$(foreach dir,$(LIBDIRS),-I$(dir)/include) \
					-I$(SDL_INCLUDE) \
					-I$(CURDIR)/$(BUILD) \
					-I$(LIBOGC_INC)

#---------------------------------------------------------------------------------
# build a list of library paths
#---------------------------------------------------------------------------------
export LIBPATHS	:= -L$(LIBOGC_LIB) $(foreach dir,$(LIBDIRS),-L$(dir)/lib)

export OUTPUT	:= $(CURDIR)/$(TARGETDIR)/$(TARGET)
.PHONY: $(BUILD) clean

#---------------------------------------------------------------------------------
$(BUILD):
	@[ -d $@ ] || mkdir -p $@
	@[ -d $(TARGETDIR) ] || mkdir -p $(TARGETDIR)
	@make --no-print-directory -C $(BUILD) -f $(CURDIR)/Makefile.gc

#---------------------------------------------------------------------------------
clean:
	@echo clean ...
	@rm -fr $(BUILD) $(OUTPUT).elf $(OUTPUT).dol

fuck:
	@echo $(INCLUDE)

#---------------------------------------------------------------------------------
run:
	psoload $(OUTPUT).dol

#---------------------------------------------------------------------------------
reload:
	psoload -r $(OUTPUT).dol


#---------------------------------------------------------------------------------
else

DEPENDS	:=	$(OFILES:.o=.d)

#---------------------------------------------------------------------------------
# main targets
#---------------------------------------------------------------------------------
$(OUTPUT).dol: $(OUTPUT).elf
$(OUTPUT).elf: $(OFILES)

$(OFILES_SOURCES) : $(HFILES)

#---------------------------------------------------------------------------------
# This rule links in binary data with these extensions: ttf lang png pcm
#---------------------------------------------------------------------------------
%.ttf.o %_ttf.h : %.ttf
	@echo $(notdir $<)
	$(bin2o)
	
%.lang.o %_lang.h : %.lang
	@echo $(notdir $<)
	$(bin2o)

%.png.o %_png.h : %.png
	@echo $(notdir $<)
	$(bin2o)
	
%.pcm.o %_pcm.h : %.pcm
	@echo $(notdir $<)
	$(bin2o)

-include $(DEPENDS)

#---------------------------------------------------------------------------------
endif
#---------------------------------------------------------------------------------


# DEBUG := 0
# BUILDASCPP := 0
# BINPREFIX :=
# CC := gcc
# CXX=$(BINPREFIX)$(CC)

# STRIPBIN=$(BINPREFIX)strip
# WINDRES=windres
# SDLCONFIG=sdl2-config
# SRC=../../src
# RSRC=../../rsrc
# OBJ=obj

# USE_OPENGL=0

# SINGLE_EPISODE=0

# OBJECTS=$(OBJ)/cvorticons.o \
#         $(OBJ)/cvorticons_memdata.o \
#         $(OBJ)/cvorticons_engine.o \
#         $(OBJ)/cvorticons_gfx.o \
#         $(OBJ)/cvorticons_input.o \
#         $(OBJ)/cvorticons_gui.o \
# 	$(OBJ)/lz.o \
# 	$(OBJ)/CRLE.o \
# 	$(OBJ)/Cunlzexe.o \
# 	$(OBJ)/imageRLE.o \
# 	$(OBJ)/chocolate-keen.o

# ifeq ($(SINGLE_EPISODE),$(filter $(SINGLE_EPISODE),0 1))
# 	OBJECTS+= $(OBJ)/cvorticons1.o \
# 	          $(OBJ)/cvorticons1_memdata.o \
# 	          $(OBJ)/cvorticons1_engine.o
# endif
# ifeq ($(SINGLE_EPISODE),$(filter $(SINGLE_EPISODE),0 2))
# 	OBJECTS+= $(OBJ)/cvorticons2.o \
# 	          $(OBJ)/cvorticons2_memdata.o \
# 	          $(OBJ)/cvorticons2_engine.o
# endif
# ifeq ($(SINGLE_EPISODE),$(filter $(SINGLE_EPISODE),0 3))
# 	OBJECTS+= $(OBJ)/cvorticons3.o \
# 	          $(OBJ)/cvorticons3_memdata.o \
# 	          $(OBJ)/cvorticons3_engine.o
# endif

# INTCXXFLAGS :=

# ifeq ($(DEBUG),1)
# 	INTCXXFLAGS+= -g -DCHOCOLATE_KEEN_CONFIG_DEBUG
# else
# 	INTCXXFLAGS+= -O2
# endif

# INTCXXFLAGS+= -I$(SRC) `$(SDLCONFIG) --cflags`
# INTLDFLAGS=`$(SDLCONFIG) --libs`

# INTCXXFLAGS+= -std=c99

# ifeq ($(SINGLE_EPISODE),0)
# 	INTCXXFLAGS+= -DCHOCOLATE_KEEN_IS_EPISODE1_ENABLED -DCHOCOLATE_KEEN_IS_EPISODE2_ENABLED -DCHOCOLATE_KEEN_IS_EPISODE3_ENABLED
# else
# 	INTCXXFLAGS+= -DCHOCOLATE_KEEN_CONFIG_SPECIFIC_EPISODE=$(SINGLE_EPISODE) -DCHOCOLATE_KEEN_IS_EPISODE$(SINGLE_EPISODE)_ENABLED
# endif


# ifeq ($(SINGLE_EPISODE),0)
# 	EXE_PATH=chocolate-keen$(EXT)
# else
# 	EXE_PATH=chocolate-keen$(SINGLE_EPISODE)$(EXT)
# endif

# .PHONY: all game clean veryclean

# all: game

# game: $(EXE_PATH)

# PRELOAD_FILE=

# $(EXE_PATH): $(OBJECTS)
# 	$(CXX) $(OBJECTS) $(LDFLAGS) $(INTLDFLAGS) -o $@ $(PRELOAD_FILES)

# $(EXE_PATH): $(OBJECTS)

# $(OBJ)/chocolate-keen_icon.o: $(RSRC)/chocolate-keen.rc
# 	$(WINDRES) $< -O coff -o $@

# $(OBJ)/lz.o: $(SRC)/decompression/CGenius/fileio/lz.c
# 	$(CXX) -c $(INTCXXFLAGS) $(CXXFLAGS) $< -o $@
# $(OBJ)/CRLE.o: $(SRC)/decompression/CGenius/fileio/compression/CRLE.c
# 	$(CXX) -c $(INTCXXFLAGS) $(CXXFLAGS) $< -o $@
# $(OBJ)/Cunlzexe.o: $(SRC)/decompression/CGenius/fileio/compression/Cunlzexe.c
# 	$(CXX) -c $(INTCXXFLAGS) $(CXXFLAGS) $< -o $@
# $(OBJ)/imageRLE.o: $(SRC)/decompression/imageRLE.c
# 	$(CXX) -c $(INTCXXFLAGS) $(CXXFLAGS) $< -o $@
# $(OBJ)/%.o: $(SRC)/%.c
# 	$(CXX) -c $(INTCXXFLAGS) $(CXXFLAGS) $< -o $@

# clean:
# 	-rm -rf $(OBJ)
# 	-rm -rf GAMEDATA
# 	-rm -f chocolate-keen