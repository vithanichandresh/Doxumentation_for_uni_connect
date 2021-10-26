# Qr code generator for section

"generate_qr.dart" Have a class using which instructor will get QR code of session. The instructor can copy the link of the QR image.

## Installation
You will have to put attend_corse_helper.dart inside your lib directory.

Install [qr_flutter](https://pub.dev/packages/qr_flutter/install)

```bash
dependencies:
  qr_flutter: ^4.0.0
```

## Session detail page

```dart
import 'package:uni_connect_student/ui/common/generate_qr.dart';

# returns 'by click generate qr code widget get visible'
Visibility(
  visible: isClickedGenerate,
  child: GetQrCodeWidgetForSection(
    sectionId: sectionId,
  ),
)

```

## GetQrCodeWidgetForSection Widget Class
This method is used to get QR code of section
```dart
import 'package:firebase_storage/firebase_storage.dart';
import 'package:qr_flutter/qr_flutter.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

GlobalKey globalKey = GlobalKey();
final String? sectionId;

GetQrCodeWidgetForSection({Key? key, this.sectionId}) : super(key: key);

# returns 'QR code image view'
# QrImage is widget from third party "qr_flutter"
@override
  Widget build(BuildContext context) {
	return RepaintBoundary(
  		key: globalKey,
	  	child: QrImage(
    	data: sectionId ?? '',
	    version: QrVersions.auto,
    	size: 200.0,
	  ),
	);
}
```
## generateQrCodeLink
This methods are used to generate link of QR Image
```dart
ValueNotifier<String> qrCodeShareLink = ValueNotifier<String>('');

# this method converts qr image widget into file.
# Also it is calling another method "uploadFile" for get Link.
Future generateQrCodeLink(BuildContext context) async {
  return await Future.delayed(Duration(seconds: 1)).then((value) async {
    RenderRepaintBoundary? boundary = globalKey.currentContext?.findRenderObject() as RenderRepaintBoundary?;
    ui.Image image = await boundary!.toImage();
    ByteData? byteData = await image.toByteData(format: ui.ImageByteFormat.png);
    var capturedBytes = byteData!.buffer.asUint8List();
    File file = File.fromRawPath(capturedBytes);
    uploadFile(file).then((value) async {
      qrCodeShareLink.value = value;
    });
  });
}

# This method is get used to upload file in firebase and get download link.
Future<String> uploadFile(File? file) async {
  if (file == null) {
    return '';
  }
  String fileName = file.path.split('/').last;
  Reference reference = FirebaseStorage.instance.ref().child(fileName);
  UploadTask uploadTask = reference.putFile(file);
  return uploadTask.then((TaskSnapshot storageTaskSnapshot) {
    return storageTaskSnapshot.ref.getDownloadURL();
  }, onError: (e) {
    throw Exception(e.toString());
  });
}


```