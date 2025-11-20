# Makefile for macOS version

# The ULMO compression used in DMG target is only supported from 10.15
# So we might aswell compile for that version
TARGET=x86_64-apple-macos10.15

BUILD_NR=$(shell git show --no-patch --format='%cs').yth.$(shell git rev-parse --short HEAD)
BUILD_NR_TARGET=$(BUILD_NR).$(TARGET)

# Sources
PROJECT_ROOT=../SIDFactoryII
SOURCE=$(PROJECT_ROOT)/source
SRC_TMP=$(PROJECT_ROOT)/main.cpp $(shell find $(SOURCE) -name "*.cpp")
SRC=$(patsubst %miniz_tester.cpp,,$(SRC_TMP))
DOCS=../dist/documentation

# Artifacts
APP_NAME=SIDFactoryII
ARTIFACTS_FOLDER=artifacts
EXE=$(ARTIFACTS_FOLDER)/$(APP_NAME)
APP=$(ARTIFACTS_FOLDER)/$(APP_NAME).app
DMG=$(ARTIFACTS_FOLDER)/$(APP_NAME)_$(BUILD_NR).dmg
DMG_CONTENTS=$(ARTIFACTS_FOLDER)/dmg
DMG_SIZE=25600
ICONSET=$(ARTIFACTS_FOLDER)/$(APP_NAME).iconset
ICONS=$(ARTIFACTS_FOLDER)/$(APP_NAME).icns

CC=clang++

CC_FLAGS= \
  -I$(SOURCE) \
	-I./App/Contents/Frameworks/SDL2.framework/Headers \
	-D_BUILD_NR=\"$(BUILD_NR_TARGET)\" \
	-F./App/Contents/Frameworks \
	-std=gnu++14 \
	-stdlib=libc++ \
	-O2 \
	-DNDEBUG \
	-flto \
	-target $(TARGET)

LINKER_FLAGS=\
	-framework SDL2 \
	-framework ApplicationServices \
	-F./App/Contents/Frameworks \
	-lstdc++ \
	-flto \
	-Wl,-rpath,@executable_path/../Frameworks \
	-target $(TARGET)

# Rule to compile .o from .cpp
%.o: %.cpp
	$(CC) $(CC_FLAGS) -c $< -o $@

# Rule to compile .o from .c
%.o: %.c
	$(CC) -target $(TARGET) -c $< -o $@

# Determine all .o files to be built
OBJ = $(SRC:.cpp=.o) $(SOURCE)/libraries/miniz/miniz.o

# Compile executable
$(EXE): $(OBJ) $(ARTIFACTS_FOLDER)
	$(CC) $(LINKER_FLAGS) -o $(EXE) $(OBJ)
	strip $(EXE)

# Make universal binary
.PHONY: universal
universal: | clean $(EXE)
	rm ${OBJ} || true
	mv $(EXE) $(EXE)_x86
	make TARGET=arm64-apple-macos11
	mv $(EXE) $(EXE)_arm
	lipo -create -output $(EXE) $(EXE)_x86 $(EXE)_arm
	rm $(EXE)_x86 $(EXE)_arm

.PHONY: app
app: $(APP)

.PHONY: dmg
dmg: $(DMG)

# ---------------------------------------------------

# Package executable and dependencies in macOS App
$(APP): universal $(ICONS)
	rm -rf $(APP)
	mkdir -p $(APP)
	cp -r App/ ${APP}
	mkdir -p ${APP}/Contents/MacOS
	mkdir -p ${APP}/Contents/Resources
	mkdir -p ${APP}/Contents/Resources/config
	mv $(EXE) $(APP)/Contents/MacOS
	cp -r $(PROJECT_ROOT)/drivers $(APP)/Contents/Resources
	cp -r $(PROJECT_ROOT)/color_schemes $(APP)/Contents/Resources
	cp -r $(PROJECT_ROOT)/config.ini $(APP)/Contents/Resources/config/
	mv $(ICONS) $(APP)/Contents/Resources
	gsed -i 's/{VERSION}/$(BUILD_NR)/g' $(APP)/Contents/Info.plist

# Collect the contents of the DMG
$(DMG_CONTENTS): $(APP) 
	rm -rf $(DMG_CONTENTS)
	mkdir -p $(DMG_CONTENTS)/$(APP_NAME)
	mv $(APP) $(DMG_CONTENTS)
	cp -r $(DOCS)/ $(DMG_CONTENTS)/$(APP_NAME)/documentation
	cp $(PROJECT_ROOT)/COPYING $(DMG_CONTENTS)
	cp -r Install.txt $(DMG_CONTENTS)
	cp -r $(PROJECT_ROOT)/music/ ${DMG_CONTENTS}/$(APP_NAME)/music
	cp -r $(PROJECT_ROOT)/drivers ${DMG_CONTENTS}/$(APP_NAME)/drivers
	ln -s /Applications $(DMG_CONTENTS)/Applications

$(DMG): $(DMG_CONTENTS)
	hdiutil create -srcfolder "$<" -volname "$(APP_NAME)" -fs HFS+ -fsargs "-c c=64,a=16,e=16" -format ULMO -imagekey -zlib-level=9 -size $(DMG_SIZE)k -ov $@

# https://developer.apple.com/library/archive/documentation/GraphicsAnimation/Conceptual/HighResolutionOSX/Optimizing/Optimizing.html
$(ICONS): ../resources/logo.svg $(ARTIFACTS_FOLDER)
	mkdir -p $(ICONSET)
	rsvg-convert -w 16 -h 16 -f png -o $(ICONSET)/icon_16x16.png $<
	rsvg-convert -w 32 -h 32 -f png -o $(ICONSET)/icon_32x32.png $<
	rsvg-convert -w 128 -h 128 -f png -o $(ICONSET)/icon_128x128.png $<
	rsvg-convert -w 256 -h 256 -f png -o $(ICONSET)/icon_256x256.png $<
	rsvg-convert -w 512 -h 512 -f png -o $(ICONSET)/icon_512x512.png $<
	rsvg-convert -w 1024 -h 1024 -f png -o $(ICONSET)/icon_512x512@2x.png $<
	cp $(ICONSET)/icon_32x32.png $(ICONSET)/icon_16x16@2x.png
	cp $(ICONSET)/icon_256x256.png $(ICONSET)/icon_128x128@2x.png
	cp $(ICONSET)/icon_512x512.png $(ICONSET)/icon_256x256@2x.png
	iconutil -c icns -o $@ $(ICONSET)
	rm -rf $(ICONSET)

$(ARTIFACTS_FOLDER):
	mkdir -p $@

.PHONY: clean
clean:
	rm ${OBJ} || true
	rm -rf $(ARTIFACTS_FOLDER)
