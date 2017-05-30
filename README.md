# run_analysis.R
rm(list=ls())
  #getting files ready, create folder and extract data
filename <- "cleaning_data.zip"
if (!file.exists(filename)){
  download.file(url = paste("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip", sep = ""),destfile = filename, mode = 'wb',cacheOK = FALSE)
}
if (!file.exists("UCI HAR Dataset")) { 
  unzip(filename) 
}
  #getting lables
activityLabels <- read.table("UCI HAR Dataset/activity_labels.txt")
activityLabels[,2] <- as.character(activityLabels[,2])
  #loading features
features <- read.table("UCI HAR Dataset/features.txt")
features[,2] <- as.character(features[,2])
keyfeatures <- grep(".*mean.*|.*std.*",features[,2])
key_names <- features[keyfeatures,2]
key_names<- gsub("-mean","Mean",key_names)
key_names <- gsub("-std","Std",key_names)
key_names <- gsub("[-()]","",key_names)

  #loading training data
train <-read.table("UCI HAR Dataset/train/X_train.txt")[keyfeatures]
trainAct <- read.table("UCI HAR Dataset/train/Y_train.txt")
trainSub <- read.table("UCI HAR Dataset/train/subject_train.txt")
train <- cbind(trainSub,trainAct,train)
  #loading testing data
test <- read.table("UCI HAR Dataset/test/X_test.txt")[keyfeatures]
testAct <- read.table("UCI HAR Dataset/test/Y_test.txt")
testSub <- read.table("UCI HAR Dataset/test/subject_test.txt")
test <- cbind(testSub, testAct, test)
  #binding
dt <- rbind(train,test)
colnames(dt) <- c("Subjects","Activities",key_names)
dt$Subjects <- as.factor(dt$Subjects)

  #processing data
library(reshape2)
  #melt data set into ids so that we can better manage it 
dt.melted <- melt(dt, id = c("Subjects","Activities"))
  #casting from a metled dataset into a new one with mean of all variables
dt.mean <- dcast(dt.melted, Subjects+Activities ~ variable, mean)
  #output txt file
write.table(dt.mean, file = "Tidydata.txt", row.names = FALSE, quote = FALSE)
