# ARO_archive_pipline
Pipline to sync data from groundhog to niagara, and then archive them into HPSS.

### NOTE:
* It will group per 1000 files(~65G) into a tarball for optimal archiving speed. 
* No checksum is included, but the scripts will compare the file number in groundhog, niagara and HPSS tarball. If the number does not match, the script will automatically repeat the previous step.

### SETUP(only do it once):
1. create a ~/.popt file and put the following line in it:
rsync alias -s -vrtlD -e "ssh"    
2. make sure you can type "ssh groundhog" in niagara and login into groundhog without passwords

#to do so:
#1. add a Host groundhog in your .ssh/config 
#2. to login without passwords, login to niagara, type: 
#cat .ssh/id_rsa.pub | ssh groundhog 'cat >> .ssh/authorized_keys'
#please google it for detailed steps

### USAGE:
1. put the folder: file4groundhog in groundhog, change the 'drivename' in check_file_info.py, run it with python3. 
   Copy the output file "$drivename_file_info.dat' back to this ARO_archive_pipline fold (same directory with the makefile).
   
   _optional: paste the other output file: drivename`_`wiki`_`doc.dat on wiki for record_
2. revise the following lines in makefile:
   * DISK=A#drive name
   * OBS_DATE=1804#observed at 2018.4
   * STARTFILE=0
   * ENDFILE=-1
#archiving folders between $STARTFILE and $ENDFILE(included) in $drivename_file_info.dat, -1 means last folder
   * NIADIR=${SCRATCH}/ARO/${OBS_DATE}/${DISK}disk/
   * ARCDIR=${ARCHIVE}/ARO/${OBS_DATE}/${DISK}disk/
#path in niagara and archive directory

3. run the following command one by one:
   * **make sync**(after ssh nia-datamover1 or 2): syncing files and create folders in dest directory (will auto run make check recursively)
#output:log_sync
   (Note: it will start a nohup process for syncing. If you would like to kill it before it finishes syncing: get the PID: ps -ef | grep "python" and kill it with "kill -9 PID")
   (Note: without syncing the note, use make sync2)
   * **make checksync**: to check if synced file number matches the one before syncing
   * **make script**: write submission script for htar
   * **make htar**: submit htar task
#htar output: htar_submit/HTAR*.OUT
   * **make checkhtar**: check the file number in HPSS tarballs
#whether there are missing files, see htar_submit/CHECK*.OUT

### MORE HELP:
* make help: will print the help 
* python rsync_ARO.py -h (with python3) will print the parameter for the syncing script.
