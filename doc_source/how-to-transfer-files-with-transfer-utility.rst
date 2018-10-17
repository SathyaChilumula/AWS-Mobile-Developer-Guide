.. Copyright 2010-2018 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. _how-to-transfer-files-with-transfer-utility:

######################################################
Transfer Files and Data Using TransferUtility and |S3|
######################################################

This page explains how to implement upload and download functionality and a number of additional storage use cases.

The examples on this page assume you have added the AWS Mobile SDK to your mobile app. To create a new cloud storage backend for your app or to add existing AWS resources, see :ref:`Add User File Storage <add-aws-mobile-user-data-storage>`.

If you use the transfer utility multipart upload feature, take advantage of automatic cleanup features by setting up the `AbortIncompleteMultipartUpload <https://docs.aws.amazon.com/AmazonS3/latest/dev/intro-lifecycle-rules.html>`__ action in your Amazon S3 bucket life cycle configuration.

If your app is built for Android and depends on long-running background transfers, see :ref:`Executing Long-running Transfers in the Background <long-running-transfers>`.


.. _how-to-transfer-utility-add-aws-user-data-storage-upload:

Upload a File
=============

.. container:: option

   Android - Java
     The following example shows how to upload a file to an |S3| bucket.

     Use :code:`AWSMobileClient` to get the :code:`AWSConfiguration` and :code:`AWSCredentialsProvider`, then create the :code:`TransferUtility` object. AWSMobileClient expects an activity context for resuming an authenticated session and creating the credentials provider.

     The following example shows using the transfer utility in the context of an Activity. If you are creating transfer utility from an application context, you can construct the CredentialsProvider and
     AWSConfiguration object and pass it into TransferUtility. The TransferUtility will check the size of file being uploaded and will automatically switch over to using multi part uploads if the file size exceeds 5 MB.

       .. code-block:: java

            import android.app.Activity;
            import android.util.Log;

            import com.amazonaws.mobile.client.AWSMobileClient;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferUtility;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferState;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferObserver;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferListener;
            import com.amazonaws.services.s3.AmazonS3Client;

            import java.io.File;

            public class YourActivity extends Activity {

                @Override
                protected void onCreate(Bundle savedInstanceState) {
                    AWSMobileClient.getInstance().initialize(this).execute();
                    uploadWithTransferUtility();
                }

                private void uploadWithTransferUtility() {

                    TransferUtility transferUtility =
                        TransferUtility.builder()
                            .context(getApplicationContext())
                            .awsConfiguration(AWSMobileClient.getInstance().getConfiguration())
                            .s3Client(new AmazonS3Client(AWSMobileClient.getInstance().getCredentialsProvider()))
                            .build();

                    TransferObserver uploadObserver =
                        transferUtility.upload(
                            "s3Folder/s3Key.txt",
                            new File("/path/to/file/localFile.txt"));

                    // Attach a listener to the observer to get state update and progress notifications
                    uploadObserver.setTransferListener(new TransferListener() {

                        @Override
                        public void onStateChanged(int id, TransferState state) {
                            if (TransferState.COMPLETED == state) {
                                // Handle a completed upload.
                            }
                        }

                        @Override
                        public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                            float percentDonef = ((float) bytesCurrent / (float) bytesTotal) * 100;
                            int percentDone = (int)percentDonef;

                            Log.d("YourActivity", "ID:" + id + " bytesCurrent: " + bytesCurrent
                                    + " bytesTotal: " + bytesTotal + " " + percentDone + "%");
                        }

                        @Override
                        public void onError(int id, Exception ex) {
                            // Handle errors
                        }

                    });

                    // If you prefer to poll for the data, instead of attaching a
                    // listener, check for the state and progress in the observer.
                    if (TransferState.COMPLETED == uploadObserver.getState()) {
                        // Handle a completed upload.
                    }

                    Log.d("YourActivity", "Bytes Transferred: " + uploadObserver.getBytesTransferred());
                    Log.d("YourActivity", "Bytes Total: " + uploadObserver.getBytesTotal());
                }
            }

   Android - Kotlin
     The following example shows how to upload a file to an |S3| bucket.

     Use :code:`AWSMobileClient` to get the :code:`AWSConfiguration` and :code:`AWSCredentialsProvider`, then create the :code:`TransferUtility` object. AWSMobileClient expects an activity context for resuming an authenticated session and creating the credentials provider.

     The following example shows using the transfer utility in the context of an Activity. If you are creating transfer utility from an application context, you can construct the CredentialsProvider and
     AWSConfiguration object and pass it into TransferUtility. The TransferUtility will check the size of file being uploaded and will automatically switch over to using multi part uploads if the file size exceeds 5 MB.

       .. code-block:: kotlin

            import android.app.Activity;
            import android.util.Log;

            import com.amazonaws.mobile.client.AWSMobileClient;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferUtility;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferState;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferObserver;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferListener;
            import com.amazonaws.services.s3.AmazonS3Client;

            import java.io.File;

            class MainActivity : Activity() {
                override fun onCreate(savedInstanceState: Bundle?) {
                    super.onCreate()
                    AWSMobileClient.getInstance().initialize(this).execute()
                    uploadWithTransferUtility(
                        "s3Folder/s3Key.txt"
                        File("/path/to/file/localfile.txt")
                    )
                }

                private fun uploadWithTransferUtility(remote: String, local: File) {
                    val txUtil = TransferUtility.builder()
                            .context(getApplicationContext)
                            .awsConfiguration(AWSMobileClient.getInstance().configuration)
                            .s3Client(AmazonS3Client(AWSMobileClient.getInstance().credentialsProvider))
                            .build()

                    val txObserver = txUtil.upload(remote, local)
                    txObserver.transferListener = object : TransferListener() {
                        override fun onStateChanged(id: Int, state: TransferState) {
                            if (state == TransferState.COMPLETED) {
                                // Handle a completed upload
                            }
                        }

                        override fun onProgressChanged(id: Int, current: Long, total: Long) {
                            val done = (((current / total) * 100.0) as Float) as Int
                            Log.d(TAG, "ID: $id, percent done = $done")
                        }

                        override fun onError(id: Int, ex: Exception) {
                            // Handle errors
                        }
                    }

                    // If you prefer to poll for the data, instead of attaching a
                    // listener, check for the state and progress in the observer.
                    if (txObserver.state == TransferState.COMPLETED) {
                        // Handle a completed upload.
                    }
                }
            }

   iOS - Swift
     The transfer utility provides methods for both single-part and multipart uploads. When a transfer uses multipart upload, the data is chunked into a number of 5 MB parts which are transferred in parallel for increased speed.

     The following example shows how to upload a file to an |S3| bucket.

       .. code-block:: swift

          func uploadData() {

             let data: Data = Data() // Data to be uploaded

             let expression = AWSS3TransferUtilityUploadExpression()
                expression.progressBlock = {(task, progress) in
                   DispatchQueue.main.async(execute: {
                     // Do something e.g. Update a progress bar.
                  })
             }

             var completionHandler: AWSS3TransferUtilityUploadCompletionHandlerBlock?
             completionHandler = { (task, error) -> Void in
                DispatchQueue.main.async(execute: {
                   // Do something e.g. Alert a user for transfer completion.
                   // On failed uploads, `error` contains the error object.
                })
             }

             let transferUtility = AWSS3TransferUtility.default()

             transferUtility.uploadData(data,
                  bucket: "YourBucket",
                  key: "YourFileName",
                  contentType: "text/plain",
                  expression: expression,
                  completionHandler: completionHandler).continueWith {
                     (task) -> AnyObject! in
                         if let error = task.error {
                            print("Error: \(error.localizedDescription)")
                         }

                         if let _ = task.result {
                            // Do something with uploadTask.
                         }
                         return nil;
                 }
          }

    The following example shows how to upload a file to an |S3| bucket using multipart uploads.

        .. code-block:: swift

          func uploadData() {

             let data: Data = Data() // Data to be uploaded

             let expression = AWSS3TransferUtilityMultiPartUploadExpression()
                expression.progressBlock = {(task, progress) in
                   DispatchQueue.main.async(execute: {
                     // Do something e.g. Update a progress bar.
                  })
             }

             var completionHandler: AWSS3TransferUtilityMultiPartUploadCompletionHandlerBlock
             completionHandler = { (task, error) -> Void in
                DispatchQueue.main.async(execute: {
                   // Do something e.g. Alert a user for transfer completion.
                   // On failed uploads, `error` contains the error object.
                })
             }

             let transferUtility = AWSS3TransferUtility.default()

             transferUtility.uploadUsingMultiPart(data:data,
                  bucket: "YourBucket",
                  key: "YourFileName",
                  contentType: "text/plain",
                  expression: expression,
                  completionHandler: completionHandler).continueWith {
                     (task) -> AnyObject! in
                         if let error = task.error {
                            print("Error: \(error.localizedDescription)")
                         }

                         if let _ = task.result {
                            // Do something with uploadTask.
                         }
                         return nil;
                 }
          }

.. _how-to-transfer-utility-add-aws-user-data-storage-download:

Download a File
===============

.. container:: option

   Android - Java
     The following example shows how to download a file from an |S3| bucket. We use :code:`AWSMobileClient` to get the :code:`AWSConfiguration` and :code:`AWSCredentialsProvider` to create the :code:`TransferUtility` object. AWSMobileClient expects an activity context for resuming an authenticated session and creating the credentials provider.

     This example shows using the transfer utility in the context of an Activity. If you are creating transfer utility from an application context, you can construct the CredentialsProvider and
     AWSConfiguration object and pass it into TransferUtility.

       .. code-block:: java

          import android.app.Activity;
          import android.util.Log;

          import com.amazonaws.mobile.client.AWSMobileClient;
          import com.amazonaws.mobileconnectors.s3.transferutility.TransferUtility;
          import com.amazonaws.mobileconnectors.s3.transferutility.TransferState;
          import com.amazonaws.mobileconnectors.s3.transferutility.TransferObserver;
          import com.amazonaws.mobileconnectors.s3.transferutility.TransferListener;
          import com.amazonaws.services.s3.AmazonS3Client;

          import java.io.File;

          public class YourActivity extends Activity {

              @Override
                protected void onCreate(Bundle savedInstanceState) {
                    AWSMobileClient.getInstance().initialize(this).execute();
                    downloadWithTransferUtility();
              }

              public void downloadWithTransferUtility() {

                  TransferUtility transferUtility =
                        TransferUtility.builder()
                              .context(getApplicationContext())
                              .awsConfiguration(AWSMobileClient.getInstance().getConfiguration())
                              .s3Client(new AmazonS3Client(AWSMobileClient.getInstance().getCredentialsProvider()))
                              .build();

                  TransferObserver downloadObserver =
                        transferUtility.download(
                              "s3Folder/s3Key.txt",
                              new File("/path/to/file/localFile.txt"));

                  // Attach a listener to the observer to get notified of the
                  // updates in the state and the progress
                  downloadObserver.setTransferListener(new TransferListener() {

                     @Override
                     public void onStateChanged(int id, TransferState state) {
                        if (TransferState.COMPLETED == state) {
                           // Handle a completed upload.
                        }
                     }

                     @Override
                     public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                           float percentDonef = ((float)bytesCurrent/(float)bytesTotal) * 100;
                           int percentDone = (int)percentDonef;

                           Log.d("MainActivity", "   ID:" + id + "   bytesCurrent: " + bytesCurrent + "   bytesTotal: " + bytesTotal + " " + percentDone + "%");
                     }

                     @Override
                     public void onError(int id, Exception ex) {
                        // Handle errors
                     }

                  });

                  // If you do not want to attach a listener and poll for the data
                  // from the observer, you can check for the state and the progress
                  // in the observer.
                  if (TransferState.COMPLETED == downloadObserver.getState()) {
                      // Handle a completed upload.
                  }

                  Log.d("YourActivity", "Bytes Transferred: " + downloadObserver.getBytesTransferred());
                  Log.d("YourActivity", "Bytes Total: " + downloadObserver.getBytesTotal());
              }
          }

   Android - Kotlin
     The following example shows how to download a file from an |S3| bucket. We use :code:`AWSMobileClient` to get the :code:`AWSConfiguration` and :code:`AWSCredentialsProvider` to create the :code:`TransferUtility` object. AWSMobileClient expects an activity context for resuming an authenticated session and creating the credentials provider.

     This example shows using the transfer utility in the context of an Activity. If you are creating transfer utility from an application context, you can construct the CredentialsProvider and
     AWSConfiguration object and pass it into TransferUtility.

       .. code-block:: kotlin

            import android.app.Activity;
            import android.util.Log;

            import com.amazonaws.mobile.client.AWSMobileClient;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferUtility;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferState;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferObserver;
            import com.amazonaws.mobileconnectors.s3.transferutility.TransferListener;
            import com.amazonaws.services.s3.AmazonS3Client;

            import java.io.File;

            class MainActivity : Activity() {
                override fun onCreate(savedInstanceState: Bundle?) {
                    super.onCreate()
                    AWSMobileClient.getInstance().initialize(this).execute()
                    downloadWithTransferUtility(
                        "s3Folder/s3Key.txt"
                        File("/path/to/file/localfile.txt")
                    )
                }

                private fun downloadWithTransferUtility(remote: String, local: File) {
                    val txUtil = TransferUtility.builder()
                            .context(getApplicationContext)
                            .awsConfiguration(AWSMobileClient.getInstance().configuration)
                            .s3Client(AmazonS3Client(AWSMobileClient.getInstance().credentialsProvider))
                            .build()

                    val txObserver = txUtil.download(remote, local)
                    txObserver.transferListener = object : TransferListener() {
                        override fun onStateChanged(id: Int, state: TransferState) {
                            if (state == TransferState.COMPLETED) {
                                // Handle a completed upload
                            }
                        }

                        override fun onProgressChanged(id: Int, current: Long, total: Long) {
                            val done = (((current / total) * 100.0) as Float) as Int
                            Log.d(TAG, "ID: $id, percent done = $done")
                        }

                        override fun onError(id: Int, ex: Exception) {
                            // Handle errors
                        }
                    }

                    // If you prefer to poll for the data, instead of attaching a
                    // listener, check for the state and progress in the observer.
                    if (txObserver.state == TransferState.COMPLETED) {
                        // Handle a completed upload.
                    }
                }
            }

   iOS - Swift
     The following example shows how to download a file from an |S3| bucket.

       .. code-block:: swift

          func downloadData() {
             let expression = AWSS3TransferUtilityDownloadExpression()
             expression.progressBlock = {(task, progress) in DispatchQueue.main.async(execute: {
                // Do something e.g. Update a progress bar.
                })
             }

             var completionHandler: AWSS3TransferUtilityDownloadCompletionHandlerBlock?
             completionHandler = { (task, URL, data, error) -> Void in
                DispatchQueue.main.async(execute: {
                // Do something e.g. Alert a user for transfer completion.
                // On failed downloads, `error` contains the error object.
                })
             }

             let transferUtility = AWSS3TransferUtility.default()
             transferUtility.downloadData(
                   fromBucket: "YourBucket",
                   key: "YourFileName",
                   expression: expression,
                   completionHandler: completionHandler
                   ).continueWith {
                      (task) -> AnyObject! in if let error = task.error {
                         print("Error: \(error.localizedDescription)")
                      }

                      if let _ = task.result {
                        // Do something with downloadTask.

                      }
                      return nil;
                  }
          }


.. _native-track-progress-and-completion-of-a-transfer:

Track Transfer Progress
=======================

.. container:: option

    Android - Java
        With the :code:`TransferUtility`, the download() and upload() methods return a :code:`TransferObserver` object. This object gives access to:

        #.  The state, as an :code:`enum`
        #.  The total bytes currently transferred
        #.  The total bytes remaining to transfer, to aid in calculating progress bars
        #.  A unique ID that you can use to keep track of distinct transfers

        Given the transfer ID, the :code:`TransferObserver` object can be retrieved from anywhere in your app, even if the app was terminated during a transfer. It also lets you create a :code:`TransferListener`, which will be updated on state or progress change, as well as when an error occurs.

        To get the progress of a transfer, call :code:`setTransferListener()` on your :code:`TransferObserver`. This requires you to implement :code:`onStateChanged`, :code:`onProgressChanged`, and :code:`onError`. For example:

        You can also query for :code:`TransferObservers` with either the :code:`getTransfersWithType(transferType)` or :code:`getTransfersWithTypeAndState(transferType, transferState)` method. You can use :code:`TransferObservers` to determine what transfers are underway, what are paused and handle the transfers as necessary.

        .. code-block:: java

            TransferObserver transferObserver = download(MY_BUCKET, OBJECT_KEY, MY_FILE);
            transferObserver.setTransferListener(new TransferListener(){

                @Override
                public void onStateChanged(int id, TransferState state) {
                    // do something
                }

                @Override
                public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                    int percentage = (int) (bytesCurrent/bytesTotal * 100);
                    //Display percentage transferred to user
                }

                @Override
                public void onError(int id, Exception ex) {
                    // do something
                }
            });

        The transfer ID can be retrieved from the :code:`TransferObserver` object that is returned from upload or download function.

        .. code-block:: java

            // Gets id of the transfer.
            int transferId = transferObserver.getId();

    Android - Kotlin
        With the :code:`TransferUtility`, the download() and upload() methods return a :code:`TransferObserver` object. This object gives access to:

        #.  The state, as an :code:`enum`
        #.  The total bytes currently transferred
        #.  The total bytes remaining to transfer, to aid in calculating progress bars
        #.  A unique ID that you can use to keep track of distinct transfers

        Given the transfer ID, the :code:`TransferObserver` object can be retrieved from anywhere in your app, even if the app was terminated during a transfer. It also lets you create a :code:`TransferListener`, which will be updated on state or progress change, as well as when an error occurs.

        To get the progress of a transfer, call :code:`setTransferListener()` on your :code:`TransferObserver`. This requires you to implement :code:`onStateChanged`, :code:`onProgressChanged`, and :code:`onError`. For example:

        You can also query for :code:`TransferObservers` with either the :code:`getTransfersWithType(transferType)` or :code:`getTransfersWithTypeAndState(transferType, transferState)` method. You can use :code:`TransferObservers` to determine what transfers are underway, what are paused and handle the transfers as necessary.

        .. code-block:: kotlin

            val transferObserver = download(MY_BUCKET, OBJECT_KEY, MY_FILE);
            transferObserver.transferListener = object : TransferListener() {
                override fun onStateChanged(id: Int, state: TransferState) {
                    // Do something
                }

                override fun onProgressChanged(id: int, current: Long, total: Long) {
                    int percent = ((current / total) * 100.0) as Int
                    // Display percent transferred
                }

                override fun onError(id: Int, ex: Exception) {
                    // Do something
                }
            }

        The transfer ID can be retrieved from the :code:`TransferObserver` object that is returned from upload or download function.

        .. code-block:: kotlin

            // Gets id of the transfer.
            val transferId = transferObserver.id;

    iOS - Swift
        To implement progress and completion actions for transfers, pass :code:`progressBlock` and :code:`completionHandler` blocks when you call :code:`AWSS3TransferUtility` to initiate the transfer.

        The following code for initiating a transfer uses the example of data upload to show how progress reporting and completion handling are typically done for all transfers. The :code:`progressBlock` gives you transfer status data you can display in the progress bar. Depending on the kind of transfer, the block is contained in either :code:`AWSS3TransferUtilityUploadExpression`, :code:`AWSS3TransferUtilityMultiPartUploadExpression or :code:`AWSS3TransferUtilityDownloadExpression.

        .. code-block:: swift

            // For example, create a progress bar
            let progressView: UIProgressView! = UIProgressView()
            progressView.progress = 0.0;

            let data = Data() // The data to upload

            let expression = AWSS3TransferUtilityUploadExpression()
            expression.progressBlock = {(task, progress) in DispatchQueue.main.async(execute: {
                    // Update a progress bar.
                    progressView.progress = Float(progress.fractionCompleted)
                })
            }

            let completionHandler: AWSS3TransferUtilityUploadCompletionHandlerBlock = { (task, error) -> Void in DispatchQueue.main.async(execute: {
                    if let error = error {
                        NSLog("Failed with error: \(error)")
                    }
                    else if(self.progressView.progress != 1.0) {
                        NSLog("Error: Failed.")
                    } else {
                        NSLog("Success.")
                    }
                })
            }

            var refUploadTask: AWSS3TransferUtilityTask?
            let transferUtility = AWSS3TransferUtility.default()
            transferUtility.uploadData(data,
                       bucket: "S3BucketName",
                       key: "S3UploadKeyName",
                       contentType: "text/plain",
                       expression: expression,
                       completionHandler: completionHandler).continueWith { (task) -> AnyObject! in
                            if let error = task.error {
                                print("Error: \(error.localizedDescription)")
                            }

                            if let uploadTask = task.result {
                                // Do something with uploadTask.
                                // The uploadTask can be used to pause/resume/cancel the operation, retrieve task specific information
                                refUploadTask = uploadTask
                            }

                            return nil;
                        }

.. _native-pause-a-transfer:

Pause a Transfer
================

.. container:: option

    Android - Java
        Transfers can be paused using the :code:`pause(transferId)` method. If your app is terminated, crashes, or loses Internet connectivity, transfers are automatically paused.

        The :code:`transferId` can be retrieved from the :code:`TransferObserver` object as described in :ref:`native-track-progress-and-completion-of-a-transfer`.

        To pause a single transfer:

        .. code-block:: java

            transferUtility.pause(idOfTransferToBePaused);

        To pause all uploads:

        .. code-block:: java

            transferUtility.pauseAllWithType(TransferType.UPLOAD);

        To pause all downloads:

        .. code-block:: java

            transferUtility.pauseAllWithType(TransferType.DOWNLOAD);

        To pause all transfers of any type:

        .. code-block:: java

            transferUtility.pauseAllWithType(TransferType.ANY);

    Android - Kotlin
        Transfers can be paused using the :code:`pause(transferId)` method. If your app is terminated, crashes, or loses Internet connectivity, transfers are automatically paused.

        The :code:`transferId` can be retrieved from the :code:`TransferObserver` object as described in :ref:`native-track-progress-and-completion-of-a-transfer`.

        To pause a single transfer:

        .. code-block:: kotlin

            transferUtility.pause(idOfTransferToBePaused);

        To pause all uploads:

        .. code-block:: kotlin

            transferUtility.pauseAllWithType(TransferType.UPLOAD);

        To pause all downloads:

        .. code-block:: kotlin

            transferUtility.pauseAllWithType(TransferType.DOWNLOAD);

        To pause all transfers of any type:

        .. code-block:: kotlin

            transferUtility.pauseAllWithType(TransferType.ANY);

    iOS - Swift
        To pause or suspend a transfer, retain references to :code:`AWSS3TransferUtilityUploadTask`, :code:`AWSS3TransferUtilityMultiPartUploadTask` or :code:`AWSS3TransferUtilityDownloadTask` .

        As described in the previous section :ref:`native-track-progress-and-completion-of-a-transfer`, the variable :code:`refUploadTask` is a reference to the :code:`UploadTask` object that is retrieved from the :code:`continueWith` block of an upload operation that is invoked through :code:`transferUtility.uploadData`.

        To pause a transfer, use the :code:`suspend` method:

        .. code-block:: swift

            refUploadTask.suspend()

.. _native-resume-a-transfer:

Resume a Transfer
=======================

.. container:: option

    Android - Java
        In the case of a loss in network connectivity, transfers will automatically resume when network connectivity is restored. If the app crashed or was terminated by the operating system, transfers can be resumed with the :code:`resume(transferId)` method.

        The :code:`transferId` can be retrieved from the :code:`TransferObserver` object as described in :ref:`native-track-progress-and-completion-of-a-transfer`.

        To resume a single transfer:

        .. code-block:: java

            transferUtility.resume(idOfTransferToBeResumed);

        To resume all uploads:

        .. code-block:: java

            transferUtility.resumeAllWithType(TransferType.UPLOAD);

        To resume all downloads:

        .. code-block:: java

            transferUtility.resumeAllWithType(TransferType.DOWNLOAD);

        To resume all transfers of any type:

        .. code-block:: java

            transferUtility.resumeAllWithType(TransferType.ANY);

    Android - Kotlin
        In the case of a loss in network connectivity, transfers will automatically resume when network connectivity is restored. If the app crashed or was terminated by the operating system, transfers can be resumed with the :code:`resume(transferId)` method.

        The :code:`transferId` can be retrieved from the :code:`TransferObserver` object as described in :ref:`native-track-progress-and-completion-of-a-transfer`.

        To resume a single transfer:

        .. code-block:: kotlin

            transferUtility.resume(idOfTransferToBeResumed);

        To resume all uploads:

        .. code-block:: kotlin

            transferUtility.resumeAllWithType(TransferType.UPLOAD);

        To resume all downloads:

        .. code-block:: kotlin

            transferUtility.resumeAllWithType(TransferType.DOWNLOAD);

        To resume all transfers of any type:

        .. code-block:: kotlin

            transferUtility.resumeAllWithType(TransferType.ANY);

    iOS - Swift
        To resume an upload or a download operation, retain references to :code:`AWSS3TransferUtilityUploadTask`, :code:`AWSS3TransferUtilityMultiPartUploadTask` or :code:`AWSS3TransferUtilityDownloadTask`.

        As described in the previous section :ref:`native-track-progress-and-completion-of-a-transfer`, the variable :code:`refUploadTask` is a reference to the :code:`UploadTask` object that is retrieved from the :code:`continueWith` block of an upload operation that is invoked through :code:`transferUtility.uploadData`.

        To resume a transfer, use the :code:`resume` method:

        .. code-block:: swift

            refUploadTask.resume()

.. _native-cancel-a-transfer:

Cancel a Transfer
=================

.. container:: option

    Android - Java
        To cancel an upload, call cancel() or cancelAllWithType() on the :code:`TransferUtility` object.

        The :code:`transferId` can be retrieved from the :code:`TransferObserver` object as described in :ref:`native-track-progress-and-completion-of-a-transfer`.

        To cancel a single transfer, use:

        .. code-block:: java

            transferUtility.cancel(idToBeCancelled);

        To cancel all transfers of a certain type, use:

        .. code-block:: java

            transferUtility.cancelAllWithType(TransferType.DOWNLOAD);

    Android - Kotlin
        To cancel an upload, call cancel() or cancelAllWithType() on the :code:`TransferUtility` object.

        The :code:`transferId` can be retrieved from the :code:`TransferObserver` object as described in :ref:`native-track-progress-and-completion-of-a-transfer`.

        To cancel a single transfer, use:

        .. code-block:: kotlin

            transferUtility.cancel(idToBeCancelled);

        To cancel all transfers of a certain type, use:

        .. code-block:: kotlin

            transferUtility.cancelAllWithType(TransferType.DOWNLOAD);

    iOS - Swift
        To cancel an upload or a download operation, retain references to :code:`AWSS3TransferUtilityUploadTask`, :code:`AWSS3TransferUtilityMultiPartUploadTask` and :code:`AWSS3TransferUtilityDownloadTask`.

        As described in the previous section :ref:`native-track-progress-and-completion-of-a-transfer`, the variable :code:`refUploadTask` is a reference to the :code:`UploadTask` object that is retrieved from the :code:`continueWith` block of an upload operation that is invoked through :code:`transferUtility.uploadData`.

        To cancel a transfer, use the :code:`cancel` method:

        .. code-block:: swift

           refUploadTask.cancel()


.. _native-background-transfers:

Background Transfers
====================

The SDK supports uploading to and downloading from Amazon S3 while your app is running in the background.

.. container:: option

    Android - Java
       No additional work is needed to use this feature. As long as your app is present in the background a transfer that is in progress will continue.

    Android - Kotlin
       No additional work is needed to use this feature. As long as your app is present in the background a transfer that is in progress will continue.

    iOS - Swift
        **Manage Transfers with the App in the Foreground, in the Background, or Suspended**

         All transfers performed by :code:`TransferUtility` for iOS happen in the background using :code:`NSURLSession` background sessions. Once a transfer is initiated, it will continue regardless of whether the initiating app moves to the foreground, moves to the background, is suspended, or is terminated by the system (transfers initiated by apps the user terminates are cancelled).

         To wake an app that is suspended or in the background when a transfer it has initiated is completed, implement the :code:`handleEventsForBackgroundURLSession` method in the :code:`AppDelegate` and have it call the :code:`interceptApplication` method of :code:`AWSS3TransferUtility` as follows.

        .. code-block:: swift

            func application(_ application: UIApplication, handleEventsForBackgroundURLSession identifier: String, completionHandler: @escaping () -> Void) {
                // Store the completion handler.
                AWSS3TransferUtility.interceptApplication(application, handleEventsForBackgroundURLSession: identifier, completionHandler: completionHandler)
            }

        **Managing Transfers When an App Restarts**

        When an app that has initiated a transfer is terminated by the system restarts, the transfer may still be in progress or may have completed. To make the restarting app aware of the status of transfers it initiated before termination, instantiate the transfer utility using the :code:`AWSS3TransferUtility.s3TransferUtility(forKey: "YOUR_KEY")` method. :code:`AWSS3TransferUtility` uses the key to uniquely identify the :code:`NSURLSession` of each transfer initiated by an app, so it is important to always use the same identifier when referring to a given transfer. :code:`AWSS3TransferUtility` will automatically reconnect to the transfers that were in progress the last time the app was running.

        Though it can be called anywhere in the app, we recommended that you instantiate :code:`AWSS3TransferUtility` when the app starts up in the :code:`appDidFinishLaunching` lifecyle method.


        **Manage a Transfer when a Suspended App Returns to the Foreground**

        When an app that has initiated a transfer becomes suspended and then returns to the foreground, the transfer may still be in progress or may have completed. In both cases, use the following code to reestablish the progress and completion handler blocks of the app.

        This code example is for downloading a file but the same pattern can be used for upload and multi-part upload.

        .. code-block:: swift

            let uploadTasks = transferUtility.getUploadTasks().result
              for task in uploadTasks! {
                   task.setCompletionHandler(completionHandler!)
                   task.setProgressBlock(progressBlock!)
            }


            let downloadTasks = transferUtility.getDownloadTasks().result
               for task in downloadTasks! {
                   task.setCompletionHandler(completionHandler!)
                   task.setProgressBlock(progressBlock!)
                }
            }


            let multiPartUploadTasks = transferUtility.getMultiPartUploadTasks().result
                for task in multiPartUploadTasks! {
                    task.setCompletionHandler(completionHandler!)
                    task.setProgressBlock(progressBlock!)

             }

.. _long-running-transfers:

Executing Long-running Transfers in the Background
==================================================


.. container:: option

    Android - Java
        Starting in version 2.7.0 of the SDK, :code:`TransferService` logic has been refactored to be compatible with recent Android changes. In version 2.7.0, this service will only monitor network connectivity changes. In previous AWS Mobile SDK versions, :code:`TransferService` was started by :code:`TransferUtility`. In v. 2.7.0, :code:`TransferService` must be started manually from your application. You can use the following code in the :code:`onCreate` method of your app's Application class to start :code:`TransferService`.

        .. code-block:: java

            getApplicationContext().startService(new Intent(getApplicationContext(), TransferService.class));

        When the network becomes offline, the in-progress transfers that are in scope of the application will be paused. When network comes back online, the transfers that are paused will be resumed.

        If you expect your app to perform long-running transfers in the background, initiate the transfers from a background service. For a recommended way to use a service to initate the transfer, see a `Transfer Utility sample application <https://github.com/awslabs/aws-sdk-android-samples/tree/master/S3TransferUtilitySample>`__ .

    Android - Kotlin
        Starting in version 2.7.0 of the SDK, :code:`TransferService` logic has been refactored to be compatible with recent Android changes. In version 2.7.0, this service will only monitor network connectivity changes. In previous AWS Mobile SDK versions, :code:`TransferService` was started by :code:`TransferUtility`. In v. 2.7.0, :code:`TransferService` must be started manually from your application. You can use the following code in the :code:`onCreate` method of your app's Application class to start :code:`TransferService`.

        .. code-block:: java

            getApplicationContext().startService(new Intent(getApplicationContext(), TransferService.class));

        When network becomes offline, the in-progress transfers that are in scope of the application will be paused. When network comes back online, the transfers that are paused will be resumed.

        If you expect your app to perform long-running transfers in the background, initiate the transfers from a background service. For a recommended way to use a service to initate the transfer, see a `Transfer Utility sample application <https://github.com/awslabs/aws-sdk-android-samples/tree/master/S3TransferUtilitySample>`__ .

    iOS - Swift
       No additional work is needed to support long-running background transfers.


.. _native-advanced-transfers:

Advanced Transfer Methods
=========================

.. contents::
   :local:
   :depth: 1

.. _native-object-metadta:

Transfer with Object Metadata
-----------------------------

.. container:: option

    Android - Java
        To upload a file with metadata, use the :code:`ObjectMetadata` object. Create a :code:`ObjectMetadata` object and add in the metadata headers and pass it to the upload function.

        .. code-block:: java

            import com.amazonaws.services.s3.model.ObjectMetadata;

            ObjectMetadata myObjectMetadata = new ObjectMetadata();

            //create a map to store user metadata
            Map<String, String> userMetadata = new HashMap<String,String>();
            userMetadata.put("myKey","myVal");

            //call setUserMetadata on our ObjectMetadata object, passing it our map
            myObjectMetadata.setUserMetadata(userMetadata);

        Then, upload an object along with its metadata:

        .. code-block:: java

            TransferObserver observer = transferUtility.upload(
              MY_BUCKET,        /* The bucket to upload to */
              OBJECT_KEY,       /* The key for the uploaded object */
              MY_FILE,          /* The file where the data to upload exists */
              myObjectMetadata  /* The ObjectMetadata associated with the object*/
            );

        To download the meta, use the S3 :code:`getObjectMetadata` method. For more information, see the `API Reference <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/s3/AmazonS3Client.html#getObjectMetadata%28com.amazonaws.services.s3.model.GetObjectMetadataRequest%29>`__.

    Android - Kotlin
        To upload a file with metadata, use the :code:`ObjectMetadata` object. Create a :code:`ObjectMetadata` object and add in the metadata headers and pass it to the upload function.

        .. code-block:: kotlin

            import com.amazonaws.services.s3.model.ObjectMetadata;

            val myObjectMetadata = new ObjectMetadata()
            myObjectMetadata.userMetadata = mapOf("myKey" to "myVal")

        Then, upload an object along with its metadata:

        .. code-block:: kotlin

            val observer = transferUtility.upload(
              MY_BUCKET,        /* The bucket to upload to */
              OBJECT_KEY,       /* The key for the uploaded object */
              MY_FILE,          /* The file where the data to upload exists */
              myObjectMetadata  /* The ObjectMetadata associated with the object*/
            )

        To download the meta, use the S3 :code:`getObjectMetadata` method. For more information, see the `API Reference <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/s3/AmazonS3Client.html#getObjectMetadata%28com.amazonaws.services.s3.model.GetObjectMetadataRequest%29>`__.

    iOS - Swift
        :code:`AWSS3TransferUtilityUploadExpression` and :code:`AWSS3TransferUtilityMultiPartUploadExpression` contain the method `setValue:forRequestHeader` where you can pass in metadata to Amazon S3.
        This example demonstrates passing in the Server-side Encryption Algorithm as a request header in uploading data to S3 using MultiPart.

        .. code-block:: swift

            let data: Data = Data() // The data to upload

            let uploadExpression = AWSS3TransferUtilityMultiPartUploadExpression()
            uploadExpression.setValue("AES256", forRequestHeader: "x-amz-server-side-encryption-customer-algorithm")
            uploadExpression.progressBlock = {(task, progress) in DispatchQueue.main.async(execute: {
                    // Do something e.g. Update a progress bar.
                })
            }

            let transferUtility = AWSS3TransferUtility.default()

            transferUtility.uploadUsingMultiPart(data:data,
                        bucket: "S3BucketName",
                        key: "S3UploadKeyName",
                        contentType: "text/plain",
                        expression: uploadExpression,
                        completionHandler: nil).continueWith { (task) -> AnyObject! in
                            if let error = task.error {
                                print("Error: \(error.localizedDescription)")
                            }

                            return nil;
                        }

.. _native-access-control-list:

Transfer with Access Control List
---------------------------------

.. container:: option

    Android - Java
        To upload a file with Access Control List, use the :code:`CannedAccessControlList` object. The `CannedAccessControlList <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/s3/model/CannedAccessControlList.html>`__ specifies the constants defining a canned access control list. For example, if you use `CannedAccessControlList.PublicRead <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/s3/model/CannedAccessControlList.html#PublicRead>`__ , this specifies the owner is granted :code:`Permission.FullControl` and the :code:`GroupGrantee.AllUsers` group grantee is granted Permission.Read access.

        Then, upload an object along with its ACL:

        .. code-block:: java

            TransferObserver observer = transferUtility.upload(
              MY_BUCKET,                          /* The bucket to upload to */
              OBJECT_KEY,                         /* The key for the uploaded object */
              MY_FILE,                            /* The file where the data to upload exists */
              CannedAccessControlList.PublicRead  /* Specify PublicRead ACL for the object in the bucket. */
            );

    Android - Kotlin
        To upload a file with Access Control List, use the :code:`CannedAccessControlList` object. The `CannedAccessControlList <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/s3/model/CannedAccessControlList.html>`__ specifies the constants defining a canned access control list. For example, if you use `CannedAccessControlList.PublicRead <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/s3/model/CannedAccessControlList.html#PublicRead>`__ , this specifies the owner is granted :code:`Permission.FullControl` and the :code:`GroupGrantee.AllUsers` group grantee is granted Permission.Read access.

        Then, upload an object along with its ACL:

        .. code-block:: kotlin

            val observer = transferUtility.upload(
              MY_BUCKET,                          /* The bucket to upload to */
              OBJECT_KEY,                         /* The key for the uploaded object */
              MY_FILE,                            /* The file where the data to upload exists */
              CannedAccessControlList.PublicRead  /* Specify PublicRead ACL for the object in the bucket. */
            )

    iOS - Swift
        To upload a file and specify permissions for it, you can use predefined grants, also known as canned ACLs. The following code shows you how to setup a file with publicRead access using the AWSS3 client.


        .. code-block:: swift

            //Create a AWSS3PutObjectRequest object and setup the content, bucketname, key on it.
            //use the .acl method to specify the ACL for the file
            let s3: AWSS3 = AWSS3.default()

            let putObjectRequest: AWSS3PutObjectRequest! = AWSS3PutObjectRequest()
            let content = "testObjectData"
            putObjectRequest.acl = AWSS3ObjectCannedACL.publicRead
            putObjectRequest.bucket = "bucket name"
            putObjectRequest.key = "file name"
            putObjectRequest.body = content
            putObjectRequest.contentLength = content.count as NSNumber
            putObjectRequest.contentType = "text/plain";

            s3.putObject(putObjectRequest, completionHandler: { (putObjectOutput:AWSS3PutObjectOutput? , error: Error? ) in
                if let output = putObjectOutput {
                    print (output)
                }

                if let error = error {
                    print (error)
                }
            })

.. _native-transfer-utility-options:

Transfer Utility Options
-------------------------

.. container:: option

    Android - Java
      You can use the :code:`TransferUtilityOptions` object to customize the operations of the :code:`TransferUtility`.

      **TransferThreadPoolSize**
      This parameter will let you specify the number of threads in the thread pool for transfers. By increasing the number of threads, you will be able to increase the number of parts of a multi-part upload that will be uploaded in parallel. By default, this is set to 2 * (N + 1), where N is the number of available processors on the mobile device. The minimum allowed value is 2.

      .. code-block:: Java

        TransferUtilityOptions options = new TransferUtilityOptions();
        options.setTransferThreadPoolSize(8);

        TransferUtility transferUtility = TransferUtility.builder()
            // Pass-in S3Client, Context, AWSConfiguration/DefaultBucket Name
            .transferUtilityOptions(options)
            .build();

      **TransferServiceCheckTimeInterval**
      The :code:`TransferUtility` monitors each on-going transfer by checking its status periodically. If a stalled transfer is detected, it will be automatically resumed by the :code:`TransferUtility`. The TransferServiceCheckTimeInterval option allows you to set the time interval
      between the status checks. It is specified in milliseconds and set to 60,000 by default.

      .. code-block:: Java

        TransferUtilityOptions options = new TransferUtilityOptions();
        options.setTransferServiceCheckTimeInterval(2 * 60 * 1000); // 2-minutes

        TransferUtility transferUtility = TransferUtility.builder()
            // Pass-in S3Client, Context, AWSConfiguration/DefaultBucket Name
            .transferUtilityOptions(options)
            .build();

    Android - Kotlin
      You can use the :code:`TransferUtilityOptions` object to customize the operations of the :code:`TransferUtility`.

      **TransferThreadPoolSize**
      This parameter will let you specify the number of threads in the thread pool for transfers. By increasing the number of threads, you will be able to increase the number of parts of a multi-part upload that will be uploaded in parallel. By default, this is set to 2 * (N + 1), where N is the number of available processors on the mobile device. The minimum allowed value is 2.

      .. code-block:: kotlin

        val options = new TransferUtilityOptions().apply {
            transferThreadPoolSize = 8
        }

        val transferUtility = TransferUtility.builder()
            // Pass-in S3Client, Context, AWSConfiguration/DefaultBucket Name
            .transferUtilityOptions(options)
            .build()

      **TransferServiceCheckTimeInterval**
      The :code:`TransferUtility` monitors each on-going transfer by checking its status periodically. If a stalled transfer is detected, it will be automatically resumed by the :code:`TransferUtility`. The TransferServiceCheckTimeInterval option allows you to set the time interval
      between the status checks. It is specified in milliseconds and set to 60,000 by default.

      .. code-block:: kotlin

        val options = new TransferUtilityOptions().apply {
            transferServiceCheckTimeInterval = 2 * 60 * 1000 // 2-minutes
        }

        val transferUtility = TransferUtility.builder()
            // Pass-in S3Client, Context, AWSConfiguration/DefaultBucket Name
            .transferUtilityOptions(options)
            .build()

    iOS - Swift
        You can use the :code:`AWSS3TransferUtilityConfiguration` object to configure the operations of the :code:`TransferUtility`.

        **isAccelerateModeEnabled**
        The :code:`isAccelerateModeEnabled` option lets you to upload and download content from a bucket that has Transfer Acceleration enabled on it. See https://docs.aws.amazon.com/AmazonS3/latest/dev/transfer-acceleration.html for information on how to enable transfer acceleration for your bucket.

        This option is set to false by default.

        .. code-block:: Swift

          //Setup credentials
          let credentialProvider = AWSCognitoCredentialsProvider(regionType: YOUR-IDENTITY-POOL-REGION, identityPoolId: "YOUR-IDENTITY-POOL-ID")

          //Setup the service configuration
          let configuration = AWSServiceConfiguration(region: .USEast1, credentialsProvider: credentialProvider)

          //Setup the transfer utility configuration
          let tuConf = AWSS3TransferUtilityConfiguration()
          tuConf.isAccelerateModeEnabled = true


          //Register a transfer utility object
          AWSS3TransferUtility.register(
              with: configuration!,
              transferUtilityConfiguration: tuConf,
              forKey: "transfer-utility-with-advanced-options"
          )


          //Look up the transfer utility object from the registry to use for your transfers.
          let transferUtility = AWSS3TransferUtility.s3TransferUtility(forKey: "transfer-utility-with-advanced-options")

        **retryLimit**
        The retryLimit option allows you to specify the number of times the TransferUtility will retry a transfer when it encounters an error during the transfer. By default, it is set to 0, which means that there will be no retries.

        .. code-block:: Swift

          tuConf.retryLimit = 5

        **multiPartConcurrencyLimit**
        The multiPartConcurrencyLimit option allows you to specify the number of parts that will be uploaded in parallel for a MultiPart upload request. By default, this is set to 5.

        .. code-block:: Swift

          tuConf.multiPartConcurrencyLimit = 3

        **timeoutIntervalForResource**
        The :code:`timeoutIntervalForResource` parameter allows you to specify the maximum duration the transfer can run.  The default value for this parameter is 50 minutes. This value is important if you use Amazon Cognito temporary credential because it aligns with the maximum span of time that those credentials are valid.

.. _native-more-transfer-examples:

More Transfer Examples
======================

.. contents::
   :local:
   :depth: 1

This section provides descriptions and abbreviated examples of the aspects of each type of transfer that are unique. For information about typical code surrounding the following snippets see :ref:`native-track-progress-and-completion-of-a-transfer`.

Downloading to a File
---------------------

The following code shows how to download an |S3| Object to a local file.

.. container:: option

    Android - Java
        .. code-block:: java

            TransferObserver downloadObserver =
                transferUtility.download(
                      "s3Folder/s3Key.txt",
                      new File("/path/to/file/localFile.txt"));

            downloadObserver.setTransferListener(new TransferListener() {

                 @Override
                 public void onStateChanged(int id, TransferState state) {
                    if (TransferState.COMPLETED == state) {
                       // Handle a completed download.
                    }
                 }

                 @Override
                 public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                       float percentDonef = ((float)bytesCurrent/(float)bytesTotal) * 100;
                       int percentDone = (int)percentDonef;

                       Log.d("MainActivity", "   ID:" + id + "   bytesCurrent: " + bytesCurrent + "   bytesTotal: " + bytesTotal + " " + percentDone + "%");
                 }

                 @Override
                 public void onError(int id, Exception ex) {
                    // Handle errors
                 }

            });

    Android - Kotlin
        .. code-block:: kotlin

            val observer = transferUtility.download(
                      "s3Folder/s3Key.txt",
                      new File("/path/to/file/localFile.txt"))
            observer.transferListener = object : TransferListener() {
                override fun onStateChanged(id: int, state: TransferState) {
                    if (state == TransferState.COMPLETED) {
                        // Handle a completed download
                    }
                }

                override fun onProgressChanged(id: Int, current: Long, total: Long) {
                    val done = ((current / total) * 100.0) as Int
                    // Do something
                }

                override fun onError(id: Int, ex: Exception) {
                    // Do something
                }
            }

    iOS - Swift
        .. code-block:: swift

            let fileURL = // The file URL of the download destination.

            let transferUtility = AWSS3TransferUtility.default()
            transferUtility.download(
                    to: fileURL
                    bucket: S3BucketName,
                    key: S3DownloadKeyName,
                    expression: expression,
                    completionHandler: completionHandler).continueWith {
                        (task) -> AnyObject! in if let error = task.error {
                            print("Error: \(error.localizedDescription)")
                        }

                        if let _ = task.result {
                            // Do something with downloadTask.
                        }
                        return nil;
                    }

Uploading Binary Data to a File
--------------------------------

.. container:: option

    Android - Java
        Use the following code to upload binary data to a file in |S3|.

        .. code-block:: java

            TransferObserver uploadObserver =
                    transferUtility.upload(
                          "s3Folder/s3Key.bin",
                          new File("/path/to/file/localFile.bin"));

            uploadObserver.setTransferListener(new TransferListener() {

                 @Override
                 public void onStateChanged(int id, TransferState state) {
                    if (TransferState.COMPLETED == state) {
                       // Handle a completed upload.
                    }
                 }

                 @Override
                 public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                       float percentDonef = ((float)bytesCurrent/(float)bytesTotal) * 100;
                       int percentDone = (int)percentDonef;

                       Log.d("MainActivity", "   ID:" + id + "   bytesCurrent: " + bytesCurrent + "   bytesTotal: " + bytesTotal + " " + percentDone + "%");
                 }

                 @Override
                 public void onError(int id, Exception ex) {
                    // Handle errors
                 }

            });

    Android - Kotlin
        Use the following code to upload binary data to a file in |S3|.

        .. code-block:: kotlin

            val observer = transferUtility.upload(
                      "s3Folder/s3Key.bin",
                      new File("/path/to/file/localFile.bin"))
            observer.transferListener = object : TransferListener() {
                override fun onStateChanged(id: int, state: TransferState) {
                    if (state == TransferState.COMPLETED) {
                        // Handle a completed download
                    }
                }

                override fun onProgressChanged(id: Int, current: Long, total: Long) {
                    val done = ((current / total) * 100.0) as Int
                    // Do something
                }

                override fun onError(id: Int, ex: Exception) {
                    // Do something
                }
            }

    iOS - Swift
        To upload a binary data to a file, you have to make sure to set the appropriate content type in the uploadData method of the TransferUtility. In the example below, we are uploading a PNG image to S3.

        .. code-block:: swift

            let data: Data = Data() // The data to upload

            let transferUtility = AWSS3TransferUtility.default()
            transferUtility.uploadData(data,
                        bucket: S3BucketName,
                        key: S3UploadKeyName,
                        contentType: "image/png",
                        expression: expression,
                        completionHandler: completionHandler).continueWith { (task) -> AnyObject! in
                            if let error = task.error {
                                print("Error: \(error.localizedDescription)")
                            }

                            if let _ = task.result {
                                // Do something with uploadTask.
                            }

                            return nil;
                        }

Downloading Binary Data to a File
---------------------------------

The following code shows how to download a binary file.

.. container:: option

    Android - Java
        .. code-block:: java

            TransferObserver downloadObserver =
                transferUtility.download(
                      "s3Folder/s3Key.bin",
                      new File("/path/to/file/localFile.bin"));

            downloadObserver.setTransferListener(new TransferListener() {

                 @Override
                 public void onStateChanged(int id, TransferState state) {
                    if (TransferState.COMPLETED == state) {
                       // Handle a completed download.
                    }
                 }
                 @Override
                 public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                       float percentDonef = ((float)bytesCurrent/(float)bytesTotal) * 100;
                       int percentDone = (int)percentDonef;

                       Log.d("MainActivity", "   ID:" + id + "   bytesCurrent: " + bytesCurrent + "   bytesTotal: " + bytesTotal + " " + percentDone + "%");
                 }

                 @Override
                 public void onError(int id, Exception ex) {
                    // Handle errors
                 }

            });

    Android - Kotlin
        .. code-block:: kotlin

            val observer = transferUtility.download(
                      "s3Folder/s3Key.bin",
                      new File("/path/to/file/localFile.bin"))
            observer.transferListener = object : TransferListener() {
                override fun onStateChanged(id: int, state: TransferState) {
                    if (state == TransferState.COMPLETED) {
                        // Handle a completed download
                    }
                }

                override fun onProgressChanged(id: Int, current: Long, total: Long) {
                    val done = ((current / total) * 100.0) as Int
                    // Do something
                }

                override fun onError(id: Int, ex: Exception) {
                    // Do something
                }
            }

    iOS - Swift
        .. code-block:: swift

            let transferUtility = AWSS3TransferUtility.default()
            transferUtility.downloadData(
                    fromBucket: S3BucketName,
                    key: S3DownloadKeyName,
                    expression: expression,
                    completionHandler: completionHandler).continueWith {
                        (task) -> AnyObject! in if let error = task.error {
                            print("Error: \(error.localizedDescription)")
                        }

                        if let _ = task.result {
                            // Do something with downloadTask.
                        }

                        return nil;
                    }


.. _transfer-encryption:

Securing your content stored in Amazon S3
=========================================

To add encryption to an S3 Object, you can follow one of the following recommended approaches:

Use Client-side Encryption
--------------------------

For uploads, you can encrypt the file locally using an algorithm of your choice and use the TransferUtility API to upload the encrypted file to S3. For downloads, you can use the :code:`TransferUtility` API to download the file and then decrypt it using the algorithm that you used to upload the file.

Use Server-side Encryption on Amazon S3
---------------------------------------

There are multiple options available for server-side encryption:

Use the Amazon S3 console to `encrypt all objects in the bucket <https://docs.aws.amazon.com/AmazonS3/latest/user-guide/default-bucket-encryption.html>`__, or to  `encrypt individual objects after they have been uploaded <https://docs.aws.amazon.com/AmazonS3/latest/user-guide/add-object-encryption.html>`__.

You can also request server-side encryption for the object being uploaded using an AWS SDK. Options include:

.. container:: option

     Android - Java
         To request server-side encryption for objects you transfer using :code:`transferUtility`, instantiate an :code:`ObjectMetadata` object, set the encryption algorythm and key value in the object's header. The following code passes an :code:`objectMetadata` instance to a :code:`transferUtility` method, using upload as an the example. The same pattern applies to download and multi-part download.

         .. code-block:: java

             TransferObserver observer = transferUtility.upload(
               MY_BUCKET,
               OBJECT_KEY,
               MY_FILE,
               myObjectMetadata
             );

         In this example, :code:`MY_BUCKET` is the bucket that is the destination of the upload; :code:`OBJECT_KEY` is the key (destination file name) for the uploaded; :code:`MY_FILE` is the source of the data to be uploaded. For more information, see :ref:`Transfer with Object Metadata <native-object-metadta>`.

         **To use AWS Key Management Service (KMS) to manage your encryption key:**

         * Set the algorithm to :code:`KMS_SERVER_SIDE_ENCRYPTION`

         * Use a KMS-generated key as the value of :code:`Headers.SERVER_SIDE_ENCRYPTION_KMS_KEY_ID`.

         .. code-block:: java

             AWSKMS kmsClient = new AWSKMSClient(AWSMobileClient.getInstance().getCredentialsProvider());
             kmsClient.setRegion(Region.getRegion(Regions.US_EAST_1));
             String kmsKey = kmsClient.createKey().getKeyMetadata().getKeyId();

             final ObjectMetadata myObjectMetadata = new ObjectMetadata();
             objectMetadata.setSSEAlgorithm(ObjectMetadata.KMS_SERVER_SIDE_ENCRYPTION);
             objectMetadata.setHeader(Headers.SERVER_SIDE_ENCRYPTION_KMS_KEY_ID, kmsKey);

         **To use your own encryption key:**

         * Set the algorithm to :code:`SERVER_SIDE_ENCRYPTION_CUSTOMER_ALGORITHM`, which causes Amazon S3 to use AES 256 encryption with an MD5 hash key.

         * Use your custom key as the value as the value of :code:`SERVER_SIDE_ENCRYPTION_CUSTOMER_KEY`.

         * Use your custom MD5 hash as the value as the value of :code:`SERVER_SIDE_ENCRYPTION_CUSTOMER_KEY_MD5`.

         .. code-block:: java`

             final ObjectMetadata objectMetadata = new ObjectMetadata();
             objectMetadata.setSSEAlgorithm(ObjectMetadata.SERVER_SIDE_ENCRYPTION_CUSTOMER_ALGORITHM);
             objectMetadata.setHeader(Headers.SERVER_SIDE_ENCRYPTION_CUSTOMER_KEY, "your_Key");
             objectMetadata.setHeader(Headers.SERVER_SIDE_ENCRYPTION_CUSTOMER_KEY_MD5, "your_md5");

     Android - Kotlin
         To request server-side encryption for objects you transfer using :code:`TransferUtility`, instantiate an :code:`ObjectMetadata` object, set the encryption algorythm and key value in the object's header. The following code passes an :code:`objectMetadata` instance to a :code:`transferUtility` method, using upload as an the example. The same pattern applies to download and multi-part download.

         .. code-block:: kotlin

             val observer = transferUtility.upload(
                    MY_BUCKET,
                      OBJECT_KEY,
                      MY_FILE,
                      myObjectMetadata
              )

         In this example, :code:`MY_BUCKET` is the bucket that is the destination of the upload; :code:`OBJECT_KEY` is the key (destination file name) for the uploaded; :code:`MY_FILE` is the source of the data to be uploaded. For more information, see :ref:`Transfer with Object Metadata <native-object-metadta>`.

         **To use AWS Key Management Service (KMS) to manage your encryption key value:**

         * Set the algorithm to :code:`KMS_SERVER_SIDE_ENCRYPTION`

         * Use a KMS-generated key as the value of :code:`Headers.SERVER_SIDE_ENCRYPTION_KMS_KEY_ID`.

         .. code-block:: kotlin

            val kmsClient = AWSKMSClient(AWSMobileClient.getInstance().getCredentialsProvider())
            kmsClient.setRegion(Region.getRegion(Regions.US_EAST_1))
            val kmsKey = kmsClient.createKey().getKeyMetadata().getKeyId()

            val myObjectMetadata = ObjectMetadata()
            objectMetadata.setSSEAlgorithm(ObjectMetadata.KMS_SERVER_SIDE_ENCRYPTION)
            objectMetadata.setHeader(Headers.SERVER_SIDE_ENCRYPTION_KMS_KEY_ID, kmsKey)

         **To use your own encryption key:**

         * Set the algorithm to :code:`SERVER_SIDE_ENCRYPTION_CUSTOMER_ALGORITHM`, which causes Amazon S3 to use AES 256 encryption with an MD5 hash key.

         * Use your custom key as the value as the value of :code:`SERVER_SIDE_ENCRYPTION_CUSTOMER_KEY`.

         * Use your custom MD5 hash as the value as the value of :code:`SERVER_SIDE_ENCRYPTION_CUSTOMER_KEY_MD5`.

         .. code-block:: kotlin`

            val objectMetadata = ObjectMetadata()
            objectMetadata.setSSEAlgorithm(ObjectMetadata.SERVER_SIDE_ENCRYPTION_CUSTOMER_ALGORITHM)
            objectMetadata.setHeader(Headers.SERVER_SIDE_ENCRYPTION_CUSTOMER_KEY, "your_Key")
            objectMetadata.setHeader(Headers.SERVER_SIDE_ENCRYPTION_CUSTOMER_KEY_MD5, "your_md5")


     iOS - Swift
        To request server-side encryption for objects you transfer using :code:`AWSS3TransferUtility`, pass an :code:`uploadExpression` instance to the :code:`AWSS3TransferUtility` transfer method. The following example uses a simple upload as its example. The pattern is the same for simple multi-part upload and download transfers.

        .. code-block:: swift

           let transferUtility = AWSS3TransferUtility.default()

           transferUtility.upload(data: data,
              bucket: "S3BucketName",
              key: "S3UploadKeyName",
              contentType: "text/plain",
              expression: uploadExpression,
              completionHandler: nil).continueWith { (task) -> AnyObject! in
               if let error = task.error {
                      print("Error: \(error.localizedDescription)")
               }

                 return nil;
              }

        For more information on how to use an transfer expression with :code:`AWSS3TransferUtility`, see :ref:`Upload files <how-to-transfer-utility-add-aws-user-data-storage-upload>` and :ref:`Download files <how-to-transfer-utility-add-aws-user-data-storage-download>` sections.

        **To configure an upload expression using Amazon Key Management System (KMS) to manage encryption key:**

        * Set the value of the expression :code:`x-amz-server-side-encryption` header to :code:`aws:kms`

        * Use your KMS key to set the value for the :code:`x-amz-server-side-encryption-aws-kms-key-id` header.

        .. code-block:: swift

            var createRequest = AWSKMSCreateKeyRequest()
            createRequest.detail = "Test Key"
            createRequest.keyUsage = AWSKMSKeyUsageTypeEncryptDecrypt
            createRequest.origin = AWSKMSOriginTypeAwsKms

            var keyMetadata: AWSKMSKeyMetadata? = nil

            (kms.createKey(createRequest).continue(withBlock: {{ t in

                var response: AWSKMSCreateKeyResponse? = t.result
                keyMetadata = response?.keyMetadata

                return nil
            }})).waitUntilFinished()

            let kmsKey = keyMetadata.keyId
                let uploadExpression = AWSS3TransferUtilityUploadExpression()
                uploadExpression.setValue("aws:kms", forRequestHeader: "x-amz-server-side-encryption")
                uploadExpression.setValue(kmsKey, forRequestHeader: "x-amz-server-side-encryption-aws-kms-key-id")
                uploadExpression.progressBlock = {(task, progress) in
                    print("Upload progress: ", progress.fractionCompleted)
                }

        **To configure an upload expression using a custom encryption key**

        * Set the value of the expression :code:`x-amz-server-side-encryption-customer-algorithm` header to :code:`AES256`

        * Use your custom key to set the value for the :code:`x-amz-server-side-encryption-customer-key` header.

        * Use your MD5 hash  to set the value for the :code:`x-amz-server-side-encryption-aws-kms-key-MD5` header.

        .. code-block:: swift

           let uploadExpression = AWSS3TransferUtilityUploadExpression()
                uploadExpression.setValue("AES256", forRequestHeader: "x-amz-server-side-encryption-customer-algorithm")
                uploadExpression.setValue("your_key", forRequestHeader: "x-amz-server-side-encryption-customer-key")
                uploadExpression.setValue("your_md5", forRequestHeader: "x-amz-server-side-encryption-customer-key-MD5")
                uploadExpression.progressBlock = {(task, progress) in
                    print("Upload progress: ", progress.fractionCompleted)
                }

.. _transfer-utility-limitations:

Limitations
===========


Long Running Transfers with Cognito Credentials
-----------------------------------------------

.. container:: option

    Android - Java
        If you expect your app to perform transfers that take longer than 50 minutes, use `AmazonS3Client <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/s3/AmazonS3Client.html>`__ instead of `TransferUtility <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/mobileconnectors/s3/transferutility/TransferUtility.html>`__.

        :code:`TransferUtility` generates Amazon S3 pre-signed URLs to use for background data transfer. Using |COG| Identity, you receive AWS temporary credentials. The credentials are valid for up to 60 minutes. Generated |S3| pre-signed URLs cannot last longer than that time. Because of this limitation, the Amazon S3 Transfer Utility enforces 50 minute transfer timeouts, leaving a 10 minute buffer before AWS temporary credentials are regenerated. After **50 minutes**, you receive a transfer failure.

    Android - Kotlin
        If you expect your app to perform transfers that take longer than 50 minutes, use `AmazonS3Client <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/s3/AmazonS3Client.html>`__ instead of `TransferUtility <http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/mobileconnectors/s3/transferutility/TransferUtility.html>`__.

        :code:`TransferUtility` generates Amazon S3 pre-signed URLs to use for background data transfer. Using |COG| Identity, you receive AWS temporary credentials. The credentials are valid for up to 60 minutes. Generated |S3| pre-signed URLs cannot last longer than that time. Because of this limitation, the Amazon S3 Transfer Utility enforces 50 minute transfer timeouts, leaving a 10 minute buffer before AWS temporary credentials are regenerated. After **50 minutes**, you receive a transfer failure.

    iOS - Swift
        When you use Amazon Cognito identity, transfers performed using the transfer utility are impacted by two different time limits:

        :code:`AWSS3TransferUtility` generates Amazon S3 pre-signed URLs to use for background data transfer that are valid for the span of time you set in the value of :code:`timeoutIntervalForResource`.

        Using Amazon Cognito, you receive AWS temporary credentials that are valid for up to 60 minutes.

        If the generated pre-signed URLs cannot last longer than the credentials, then transfers will run into errors. The default value of :code:`timeoutIntervalForResource` is set to 50 minutes to work well with Amazon Cognito Identity. For transfers that need to run longer than 50 minutes, you can setup static credentials using :code:`AWSStaticCredentialsProvider` and increase :code:`timeoutIntervalForResource` ( up to a maximum value of 7 days).

