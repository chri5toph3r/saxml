# saxml
Embedded XML Parser

![Build Status](https://github.com/zorxx/saxml/actions/workflows/cmake.yml/badge.svg)

saxml is a truly small event-driven XML parser designed for use in embedded/microcontroller applications.

Since saxml is a SAX XML parser (see https://en.wikipedia.org/wiki/Simple_API_for_XML), the parser has a very small memory footprint (there's no XML document stored on the heap). Instead, the XML document is streamed to the parser a single character at a time. As the parser encounters interesting events (such as a start tag, end tag, attribute, etc.), the parser executes callback functions which are registered by the calling application. This allows the calling application to perform application-specific operations based on XML parsing events.

The parse depth and heirarchy can easily be maintained by an application through the use of a stack; push the tag name on the stack each time a tagHandler event is handled and pop the top element off the stack each time a tagEndHandler event is handled.

See the test subdirectory for a simple example application. `include/saxml/saxml.h` includes a detailed description of the API.

saxml performs no validation of the XML document

### PlatformIO

Add to the following line to your project's platformio.ini file: ``` lib_deps = https://github.com/zorxx/saxml ```.
PlatformIO will automatically download the code and set the include path when building. The library can be used by adding ```#include "saxml/saxml.h"``` in the source files.

Example platformio.ini file:
```
[env]
platform = espressif32
framework = espidf
monitor_speed = 115200

[common]
lib_deps = https://github.com/zorxx/saxml

[env:nodemcu-32s]
board = nodemcu-32s
board_build.filesystem = littlefs
board_build.partitions = min_littlefs.csv
lib_deps = ${common.lib_deps}
```

### Example #1 (test.xml)

XML Document:
```xml
<begin  >
<second_begin>
<nothing_much/>
 content
</second_begin>
</begin>
```
Callbacks executed:

```
tagHandler: 'begin'
tagHandler: 'second_begin'
tagHandler: 'nothing_much'
tagEndHandler: 'nothing_much'
contentHandler: 'content
'
tagEndHandler: 'second_begin'
tagEndHandler: 'begin'

```

### Example #2 (test2.xml)

XML Document:
```xml
<begin  >
<second_begin yes no="hello">
<nothing_much/>content</second_begin>
more content goes here
</begin>
```

Callbacks executed:
```
tagHandler: 'begin'
tagHandler: 'second_begin'
attributeHandler: 'yes'
attributeHandler: 'no="hello"'
tagHandler: 'nothing_much'
tagEndHandler: 'nothing_much'
contentHandler: 'content'
tagEndHandler: 'second_begin'
contentHandler: 'more content goes here
'
tagEndHandler: 'begin'
```

### Example #3 (test3.xml)

XML Document:
```xml
<begin  >
   <second_begin yes no="hello">
      <nothing_much attribute_in_small_tag />
      <another_begin>
      </another_begin>
   </second_begin>
   more content goes here
</begin>
```

Callbacks executed:
```
tagHandler: 'begin'
tagHandler: 'second_begin'
attributeHandler: 'yes'
attributeHandler: 'no="hello"'
tagHandler: 'nothing_much'
attributeHandler: 'attribute_in_small_tag'
tagEndHandler: ' '
tagHandler: 'another_begin'
tagEndHandler: 'another_begin'
tagEndHandler: 'second_begin'
contentHandler: 'more content goes here
'
tagEndHandler: 'begin'
```
Note that, in this example, the tagEndHandler is called with a single-character string parameter for the empty tag (nothing_much) that contains an attribute. This is an ideosyncrasy of the SAX parser, since the empty tag's name isn't stored by the parser, in order to save heap usage.
