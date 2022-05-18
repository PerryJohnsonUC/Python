import os
import glob
import datetime
import re
import sys
from optparse import OptionParser

def main():
    global masterPath

    #options for test names and numbers
    optparser = OptionParser()
    optparser.add_option("-n", "--number", dest = "number", default = None, help = "Specify number of most recent files to keep in folder")
    optparser.add_option("-t", "--testname", dest = "testname", default = None, help = "Specify test path to clean out passed tests (absolute path)")
    optparser.add_option("-r", "--regnum", dest = "regnum", default = None, action = 'append', help = "Specify which regression number in test name to clean out passed tests")
    optparser.add_option("-l", "--listonly", dest = "listonly", action = "store_true", help = "List all regression numbers within testname folder")
    (options, args) = optparser.parse_args()

    #testname required
    if not options.testname:
        optparser.error('Testname path not given')
    else:
        masterPath = options.testname
        if not os.path.exists(masterPath):
            optparser.error('Invalid regression name')
        if not options.number and not options.regnum and not options.listonly:
            optparser.error("Too few arguments")

    #list files in folder option
    if options.listonly:
        testFolders = []
        print('Regressions in ' + masterPath + ':')
        if not masterPath.endswith('/'):
            masterPath += '/'
        masterPath = masterPath + '*'
        for folder in glob.glob(masterPath):
            folder = folder.rsplit('/',1)[1]
            testFolders.append(folder)
            testFolders.sort()
        for x in testFolders:
            print(x)
        sys.exit()

    #cannot give regnumber and number of files to keep, pick one or the other to perform specific function
    if options.number and options.regnum:
        optparser.error('Error: gave too many arguments')

    #if you want to clear build directories, you must pick a reg number to do it for
    if options.number:
        numKeep = options.number
        delFiles(numKeep, masterPath)
    if options.regnum:
        regNum = list(options.regnum)
        for x in regNum:
            rmpass(masterPath, x)
            removeSimv(x)

#deletes regressions from a testname folder
def delFiles(numKeep, masterPath):
    gitFolders = []
    arrFiles = []
    if not masterPath.endswith('/'):
        masterPath += '/'
    searchPath = masterPath + '*/'
    for folder in glob.glob(searchPath):
        gitPlace = os.popen('find ' +folder + ' -maxdepth 4 -type f -name git_report').read()
        gitPlace = gitPlace.rstrip()
        if gitPlace:
            gitFolders.append(gitPlace)
        else:
            arrFiles.append([folder, '2022-01-01'])

    for folder in gitFolders:
        f = open(folder, 'r')
        text = f.read()
        match = re.findall('\d{4}-\d+-\d+', text)
        match = ''.join(match)
        if match == '':
            match = '2022-01-01'
        arrFiles.append([folder,match])

    arrFiles = sorted(arrFiles, key = lambda x:x[0])
    arrFiles = sorted(arrFiles, key = lambda x:datetime.datetime.strptime(x[1],"%Y-%m-%d"))

    #avoid deleting too many files, might not be necessary
    confirm = input('Do you want to delete all but the ' + numKeep + ' most recent files in ' + masterPath + '? (y/n) ')
    if confirm != 'yes' or confirm != 'y':
        return
    for x in arrFiles[:-int(numKeep)]:
        tempPath = re.split('[/]*_dv',x[0])[0]
        tempPath = tempPath.rsplit('/',1)[0]
        os.system('rm -r ' + tempPath)
        #redundancy
        if os.path.exists(tempPath):
            os.system('rm -r ' + tempPath)

#removes test folders that passed from testname folder
def rmpass(masterPath, regNum):
    global saveBuild
    regPlace = ''
    saveBuild = []
    if not masterPath.endswith('/'):
        masterPath += '/'
    folderName = masterPath + regNum + #specific file location
    for folder in glob.glob(folderName):
        regPlace = os.popen('find ' + folder + ' -maxdepth 2 -type f -name reg_report').read()
        regPlace = regPlace.rstrip()
    if os.path.exists(regPlace):
        with open(regPlace) as f:
            while True:
                line = f.readline()
                if ' RUN ' in line or ' NR ' in line or ' DONE ' in line:
                    if ' PASS ' in line:
                        test = line.split()[0]
                        regPath = regPlace.split('reg_report',1)[0]
                        if os.path.exists(regPath + test):
                            os.system('rm -r ' + regPath + test)
                    if ' FAIL ' in line:
                        test = line.split()[0]
                        regPath = regPlace.split('reg_report',1)[0]
                        logPath = regPath + test + '/log'
                        if os.path.exists(logPath):
                            readLog(logPath)
                if ' TEST INFO ' in line:
                    break

def readLog(logPath):
    with open(logPath) as File:
        line = File.readline()
        line = line.split()[1]
        buildNum = re.findall('build_[0-9]+',line)
        buildNum = ''.join(buildNum)
        if buildNum not in saveBuild:
            saveBuild.append(buildNum)
        return

#removes simv files from the build directories in testname
def removeSimv(regNum):
    global masterPath
    if not masterPath.endswith('/'):
        masterPath += '/'
    buildPath = masterPath + regNum + #specific file location
    for folder in glob.glob(buildPath):
        buildIn = folder.rsplit('/',1)[1]
        if buildIn in saveBuild:
            continue
        else:
            os.system('rm -r ' + folder)
if '__main__' == __name__:
    main()
