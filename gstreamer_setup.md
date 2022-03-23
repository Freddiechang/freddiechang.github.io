# Setup Gstreamer Python Binding
## Install Gstreamer through package manager
See [here](https://gstreamer.freedesktop.org/documentation/installing/on-linux.html?gi-language=python).   
```
apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 gstreamer1.0-qt5 gstreamer1.0-pulseaudio
```   
## Use conda to install dependencies
See [here](https://lifestyletransfer.com/how-to-install-gstreamer-python-bindings/).
```
conda install -c conda-forge pyobject gobject-introspection gtk3 gst-python
```
## Configure Gstreamer plugin lib path
See [here](https://stackoverflow.com/questions/54097034/gst-good-plugins-installed-but-no-element-autovideosink).   
Run   
```
locate libgstautodetect.so
```
to get the plugin lib path. Then add the path to the environment:
```
export GST_PLUGIN_SYSTEM_PATH_1_0=$GST_PLUGIN_SYSTEM_PATH_1_0:#PATH#
```
## Verification
```
import gi
gi.require_version("Gtk", "3.0")
from gi.repository import Gtk

window = Gtk.Window(title="Hello World")
window.show()
window.connect("destroy", Gtk.main_quit)
Gtk.main()
```
This should show a window. See [here](https://pygobject.readthedocs.io/en/latest/getting_started.html).   
```
gst-launch-1.0 videotestsrc ! autovideosink
```
This should show a window with test video.   
```
gst-inspect-1.0 autovideosink
```
This should show info about autovideosink.
