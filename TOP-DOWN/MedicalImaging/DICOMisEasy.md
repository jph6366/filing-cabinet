
- DICOM is first of all an INTERFACE DEFINITION.
	- its success relies on the ability to integrate medical systems manufactured by many different vendors
- Medical imaging equipment today can immediately query the medical imaging archive (PACS) and retrieve images that were created by other systems and display them with little to no set up, and vice versa

- in DICOM there is a command called C-FIND that is used to make queries to another system, to search for images and other entities.  (instead of sql)
- DICOM C-FIND command has three variants, each with its own data model. 
	- These variants are 'Patient Root', 'Study Root', ~~'Patient Study Only Root'.~~ (obsolete)


### How to DICOMIZE a JPEG image
- read jpg
- dump headers/tags to text
``` c#
# Dicom-Data-Set  
# Used TransferSyntax: UnknownTransferSyntax  
(0008,0016) UI =SecondaryCaptureImageStorage # 26, 1 SOPClassUID  
(0028,0002) US 3 # 2, 1 SamplesPerPixel  
(0028,0004) CS [YBR_FULL_422] # 12, 1 PhotometricInterpretation  
(0028,0006) US 0 # 2, 1 PlanarConfiguration  
(0028,0010) US 96 # 2, 1 Rows  
(0028,0011) US 372 # 2, 1 Columns  
(0028,0100) US 8 # 2, 1 BitsAllocated  
(0028,0101) US 8 # 2, 1 BitsStored  
(0028,0102) US 7 # 2, 1 HighBit  
(0028,0103) US 0 # 2, 1 PixelRepresentation  
(0028,2110) CS [01] # 2, 1 LossyImageCompression  
(0028,2114) CS [ISO_10918_1] # 12, 1 LossyImageCompressionMethod  
(7fe0,0010) OB (PixelSequence #=2) # u/l, 1 PixelData  
  (fffe,e000) pi (no value available) # 0, 1 Item  
  (fffe,e000) pi ff\d8\ff\db\00\43\00\08\06\06\07\06\05\08\07\07\07\09\09\08\0a\0c... # 9624, 1 Item  
(fffe,e0dd) na (SequenceDelimitationItem) # 0, 0 SequenceDelimitationItem
```


TODO ADD IMAGE
illustration of DICOM element encoding in a DICOM data stream
(DICOM standard, Chapter 5)

1. **TAG**
   Every DICOM element that uniquely defines the element and its properties
   
   the **DICOM TAG** is comprised of two short numbers called **GROUP** and **ELEMENT**
	1. tags that are related to one another sometimes have the same group
2. **VALUE REPRESENTATION**
   The **VR** is represented as two character code and defines the data type of the element
   
   *in the dump above we can see UI for Unique Identifier, US for Unsigned Short, CS for Coded String, and OB for Other Byte i.e. a byte stream*
   
	1. The tag implicitly defines the VR, which is why the VR is usually redundant and can be omitted
	2. However, the IHE recommendation is to explicitly state the VR when serializing DICOM objects into files or into network buffers
3. **VALUE LENGTH**
   DICOM is a binary protocol and the elements have length and are always even length
	1. Even if the elements's value is a single character, the element length should be and the value will be padded by a space (ASCII 0x20)
	2. String types (CS, UI) are padded by space and binary types like US are padded by null 0x0


- There are thousands of DICOM elements (see Chapter 6, DICOM standard) from the very basic attributes of patient name and birth date to the most esoteric uses of 3D surface vortices.

**the motivation of adhering to a standard is to enable interoperability**
- By detailing **information object definitions** (IOD's) DICOM enables to exchange virtual objects between applications without knowing in advance anything about the application we are going to interface with.
	- an **IOD** is a collection of modules and a module is a collection of elements from one information entity that together represent something
		- The modules are also defined in chapter 3 of the DICOM standard in appendix C
- IOD’s are specified in Appendix A of chapter 3 of the standard

simplified view of the DICOM data model

Two object oriented concepts, **composition** and **reuse**, that are used by DICOM is the Modules that are parts shared between different IOD’s.

the DICOM network protocol makes a distinction between Normalized and Composite operations and there’s N protocol and C protocol with different commands for each one.

- All **DICOM Objects** must include the **SOP Common Module** and **modules** from the four main IE’s: 
	- **Patient, Study, Series and Image** 
	  (Image and Instance are the same in DICOM. Once there were only images but then objects that are not images has been defined and the name thus changed from Image to Instance in order to represent an instance of a SOP class).
	- All **DICOM Images,** that is **DICOM Instances** that Are **Images**, must include the Image Module. 
		- Because Every **DICOM Object** must be part of a **Series**, 
			- all **DICOM Objects** must include the **General Series Module** and because all series must be part of a **Study**, 
				- Every **DICOM Object** must include the **General Study Module** and because every study is made on some patient, 
					- all **DICOM objects** must have a **Patient Module**. 
						- You probably wonder what **SOP** means? That's an acronym for **"Service Object Pair"**
						  SOP is a pair of a DICOM Sevice and and DICOM Object like Secondary Capture Object and Storage Service.

Every entity in the DICOM Data Model also has a UID with the exception of the patient. Patients are identified using the combination of their name and ID. Studies, Series, all have UID’s. DICOM Archives (PACS) should use the UID’s to index their databases so when other applications make searches (Queries) they can refer to objects using the UID’s and the archive can respond to the searches quickly.

In "proper" DICOM integration the Study Instance UID is provided by the department IT system (RIS/PACS) through a DICOM service called Modality Worklist but devices can default to creating the Study Instance UID if it’s not provided from an external system. The Series Instance UID and SOP Instance UID are always generated by the Imaging device


  
Now let's move to DICOM files. Just like in DICOM networking, DICOM files must be read by all applications so thats a serialization too. Here are the rules for DICOM files:  
  
When writing a DICOM object to file, the application that creates the file writes it in the following way:

1. The first 128 bytes are null (0x00) 
2. Bytes 128 - 131 (zero based) are 'DICM' which is the DICOM magic number 
3. Add to the object a file meta header - a group of elements of group 0002 that are the first elements in the object. 
4. Group 0002 is written in Little Endian Explicit 
5. Element (0002,0010) is the Transfer Syntax UID that is used for all the elements other than group 2.

So, when reading a DICOM file, a DICOM library should do this:  
  

1. Read 132 bytes (these 132 bytes are called the preamble) and see that 128-131 equal "DICM" 
2. Start parsing using Little Endian Explicit all the group 0002 elements 
3. Check the value of element (0002,0010) and use this transfer syntax for the rest of the file.

Important: always remove group 0002 before sending objects over the network. Group 0002 is strictly for DICOM files.

this is how a DICOM file actually looks like in the byte level of three DICOM files of exactly the same object opened in a binary editor, each file was saved with a different transfer syntax
- Up to the highlighted part, the files are identical but for the value of the transfer syntax UID element in the file meta header. You can see the 128 0's and the DICM and then the elements of group 0002. In all three files this part is little endian explicit and you can see the VR codes UL and then OB just after the preamble.
	- ==The highlighted part is the first data element of the object itself,== which is element (0008,0005). While in the *Little Endian files* (left and right) the *bytes are ordered 08 00 05 00*, in the *Big endian file* (center) *the order is 00 08 00 05*.
	- Then, in the explicit **VR** files (left and center) the tag is followed by '**CS**' which is the **VR** code of this element. 
		- **CS** stands for **Code String** and tells us the data type of this element. 
		- In the implict VR file this code is missing. It is implicitly specified by the tag. Tags always have the same type. 
			  (this tag is called extended character set and it is the code string of the character set encoding for the strings in the file)
		- After that we have the data element length which is 0xA 
		  (meaning the value is 10 bytes long) 
		- And then the value itself 'ISO_IR 192' which means that the strings in this DICOM files are encoded using UTF-8. 
			- Note that in the **explicit VR files the length is stored in a two bytes** while in the implicit **VR file the length is stored in four bytes**
			  
			  
Recommendations as far as transfer syntax goes:  

1. Always support and propose all 3 basic transfer syntaxes: LEI, LEE and BEE
2. If possible, always prefer LEE as your default.


  
To search the PACS we use the DICOM command C-FIND. This command takes as an argument a DICOM object that represent a query. The PACS transforms the object that we send to a query, probably to SQL, runs it and then transform every result record back into a DICOM object and send it back to us in a C-FIND response. The PACS sends one C-FIND response for every result record. While still running, the status field of the C-FIND response command is pending (0xFF00). The last response has a status success. It may of course fail and then RZDCX will throw an exception with the failure reason and status. It may also succeed but with no matches (empty results set

**C-MOVE is a DICOM command that means this: The calling AE (we) ask the called AE (the PACS) to send all the DICOM Instances that match the identifier to the target AE.**

The following diagram, taken from part 2 of the DICOM standard, is commonly seen in DICOM Conformance Statements as the Data Flow diagram of the Q/R Service. These diagrams and their notation are defined by the standard in part 2 that specify the DICOM Conformance Statement – a standard document that every application vendor should provide and that describes how they implemented the standard in their product.
- The vertical dashed line represents the DICOM Protocol Interface between the two applications (it is usually a single dashed line)
- The arrows across the interface represents DICOM associations. 
	- The arrow points from the application that initiates the association (the requester) to the application that responds to it (the responder or accepter).
- The upper part of the diagram shows the control channel where the C-MOVE request is sent and statuses are reported back by the PACS.
- The lower part of the diagram shows the data channel where the DICOM instances are sent to the client.

