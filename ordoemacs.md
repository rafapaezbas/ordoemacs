# Ordoemacs.
## What is Ordoemacs?
Ordoemacs is a fictional piece of software appearing in Neal Stephenson 1999 speculative fiction novel Cryptonomicon. In the book, Epiphyte is a corporation in the mission of creating a data haven. The members of Epiphyte use their own encryption software called Ordo (abbr. for "Novus ordo seclorum"), that was created by one of them. In addition, Randall "Randy" Lawrence Waterhouse uses a modified version of Emacs called Ordoemacs, decribed like this on the novel:
> Returning to his seat, he fires up OrdoEmacs, which is a marvelously paranoid piece of software invented by John Cantrell. Emacs in its normal form is the hacker’s word processor, a text editor that offers little in the way of fancy formatting capabilities but does the basic job of editing plain text very well. Your normally cryptographically paranoid hacker would create files using Emacs and then encrypt them with Ordo later. But if you forget to encrypt them, or if your laptop gets stolen before you get a chance to, or your plane crashes and you die but your laptop is sieved out of the muck by baffled-but-dogged crash investigators and falls into the hands of federal authorities, your files can be read. For that matter it is possible even to find ghostly traces of old bits on a hard drive’s sectors even after the file has been overwritten with new data.
> OrdoEmacs, on the other hand, works exactly like regular Emacs, except that it encrypts everything before writing it out to disk. At no time is plaintext ever laid down on a disk by OrdoEmacs—the only place it exists in its plain, readable form is in the pixels on the screen, and in the volatile RAM of the computer, whence it vanishes the moment power is shut down. Not only that, but it’s coupled to a screensaver that uses the little built-in CCD camera in the laptop to check to see if you are actually there. It can’t recognize your face, but it can tell whether or not a vaguely human-shaped form is sitting in front of it, and if that vaguely human-shaped form goes away, even for a fraction of a second, it will drop into a screen-saver that will blank the display and freeze the machine until such time as you type in a password, or biometrically verify your identity through voice recognition.

Basically Ordoemacs is able to do two things:
* Encprypts every file before writing it to disk.
* Is able to check if someone is actually in front of the computer, if not, the program will drop into a screen-saver until the owner logs in again.

What a fantastic idea! The concept is so well explained and original that is it makes de reader imagination fly. The author already saw the potencial of a camera attached to a computer for user indetification. Now, this part of the plot is based on the year 1997, 23 years later face recognition would also be totally posible!
## Creating an Emacs minor mode for simuting Ordoemacs.
The flexibility of Emacs, allows to create a minor-mode (an optional mode that alters the functionality of Emacs) binded to a file extension, in this case ".ordo".
This minor mode will work for the first point mentioned before, with the help of [Gnu Privacy Guard](https://gnupg.org/), it will encrypt and decrypt files only storing the plain content in RAM. Also, it will use an usb storage for saving the encryption key.
### Installation:
1. Install gpg, openssl and emacs.
2. Mount the usb storage device in the folder /media/usb-drive:
```
mount /dev/sda /media/usb-drive
```
3. Generate random key:
```
openssl rand -base64 16 >> /media/usb-drive/passphrase.txt
```
4. Include minor mode in Emacs config.el:
```lisp
(define-minor-mode ordoemacs-mode
  "Ordoemacs mode."
  :lighter "ordoemacs"
  (add-hook 'write-contents-functions 'encrypt))

(defun encrypt ()
  "Encrypts the buffer and replaces file content"
  (let ((output-path "/dev/shm/w_temp"))
    (write-region nil nil output-path)
    (shell-command (concat "gpg --batch --passphrase-file /media/usb-drive/passphrase.txt -c " output-path))
    (shell-command (concat "rm " (buffer-file-name)))
    (shell-command (concat "mv /dev/shm/w_temp.gpg " (buffer-file-name)))
    (shell-command (concat "rm " output-path))
    (set-buffer-modified-p nil) ;; set the buffer as unmodified
    t)) ;; return non-nil value to abort default save


(defun decrypt ()
  "Checks if file exists, if it does, decrypts it and inserts the content to the buffer."
  (when (file-exists-p (buffer-file-name))
    (let ((output-path "/dev/shm/temp"))
      (shell-command (concat "gpg --batch --passphrase-file /media/usb-drive/passphrase.txt -o " output-path " " (buffer-file-name)))
      (insert-file-contents output-path  nil nil nil t)
      (shell-command "rm /dev/shm/temp"))))

(defun enable-ordoemacs-mode ()
  "Enables ordoemacs mode and decrypts file."
  (progn
    (ordoemacs-mode +1)
    (decrypt)))

(defun disable-ordoemacs-mode ()
  "Disables ordo emacs mode."
  (progn
    (ordoemacs-mode 0)
    (remove-hook 'write-contents-functions 'encrypt)))

;;; Every time a file is open, it enables or disables ordoemacs-mode
(add-hook 'find-file-hook
          (lambda ()
            (if (string= (file-name-extension buffer-file-name) "ordo")
              (enable-ordoemacs-mode)
              (disable-ordoemacs-mode))))
```
#### What is /dev/shm?
`/dev/shm` is a temporary that uses RAM normally functioning as a shared memory space for inter process communication.
### Usage
Create a file with extension ".ordo". After save, the `encrypt`  function will execute, encrypting the file using `/dev/shm` and storing the content in the buffer file location. In order to use it, the usb-storage must be mounted.
## Some conclusions.
I am very impressed with the power of the Emacs editor. Without serious Lisp experience, in some hours I have been able to put together a pretty decent implementation of Ordoemacs.
On the other side, even though I see the power of Lisp, I cannot ignore how primitive the syntax is, making the language to difficult to use, at least from my perspective.
Neal Stephenson is one of the greatest of our time. His books a full of breaking concepts and the writing is just fenomenal, do not miss any of his books!

