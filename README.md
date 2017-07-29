# MPhil-Thesis-Vaka-Valsdottir-Matlab-code

participant = input('Participant:');
session = input('Session based on experimental procedure (might included multiple sessions):');
session_lookingtimes = input('Session based on experimental design:');
EEG_delay = input('Please enter the Video-EEG delay (s): ');

%--- reading in eeg data ----

eegfilename = strcat(num2str(participant),'_','session',num2str(session),'_EEG_matlab_babble','.mat');
load(eegfilename);

%----testing whether data is missing----

sample_times = data(:,34);

data_droppage = diff(sample_times);

if any(data_droppage > 1)
    disp('EEG data is missing')
else
    disp('All EEG data accounted for')
end


%--- reading in looking times from coding

videofilename = strcat('looking_babble_',num2str(participant),'_s',num2str(session_lookingtimes),'.txt');
load(videofilename);
[Video_Start_time,Video_End_time] = textread(videofilename,'%f %f');

%---- reading in the corrisponding audiofiles------

participantnumber = str2num(participant);
type1 = [1:3:36];
type2 = [2:3:36];
type3 = [3:3:36];
if  any(participantnumber == type1)
    load('type_1_stimuli_audio.mat')
elseif any(participantnumber == type2)
   load('type_2_stimuli_audio.mat')
elseif any(participantnumber == type3)
    load('type_3_stimuli_audio.mat')
end
  
%---- Variables used for calulations

Start_time = Video_Start_time - EEG_delay;
End_time = Video_End_time - EEG_delay;
EEG_diff = End_time - Start_time;

%-----
% Extracting the start times for familiarizations 
% Get_Familiarization_Times provided by Stani Stanimira Georgieva 

fam_eeg = Get_Familiarization_Times(participant,session, EEG_delay);

%-------

% Extracting the start and end times for each of the presentation videos
% and eeg times --- sound {1=vidoes;2=eeg,number of familiarization}(i=look,1=start_time;2=end_time)

% EEG       

        for v = 1:3
            for cond = 1:3
        test = Start_time >= fam_eeg(v,cond)-2 & Start_time <= fam_eeg(v,cond)+21;
        tmp = test.*Start_time; tmp(tmp==0)=[];
        tmp1 = test.*End_time; tmp1(tmp1==0)=[];
        eeglook{v,cond} = [tmp, tmp1]; clear test tmp tmp1
            end
        end
        
        
        for v = 1:3
            for cond = 1:3        
        FSamp = 500;
        for i = 1:size(eeglook{v,cond},1)
        ep_extract{v,cond}{i} = data(eeglook{v,cond}(i,1)*FSamp:eeglook{v,cond}(i,2)*FSamp,:);
        end
            end
        end
          
% VIDEO
        
        for v = 1:3
            for cond = 1:3
        test = Start_time >= fam_eeg(v,cond)-2 & Start_time <= fam_eeg(v,cond)+21;
        tmp = test.*(Start_time-fam_eeg(v,cond)); tmp(tmp==0)=[];
        tmp1 = test.*(End_time-fam_eeg(v,cond)); tmp1(tmp1==0)=[];
        tmp( find(tmp<0) ) = 0;
        videolook{v,cond} = [tmp, tmp1]; clear test tmp tmp1
            end
        end
        
 for v = 1:3
            for cond = 1:3        
        FSamp = 500;
        for i = 1:size(videolook{v,cond},1)
            if videolook{v,cond}(i,2)*FSamp < length(video{v,cond})
                famvideo_extract{v,cond}{i} = video{v,cond}(videolook{v,cond}(i,1)*FSamp:videolook{v,cond}(i,2)*FSamp,:);            
            else
                famvideo_extract{v,cond}{i} = video{v,cond}(videolook{v,cond}(i,1)*FSamp:length(video{v,cond}),:);
                
            end
            end
        end
 end
 

save(strcat(participant, '_', session_lookingtimes, 'eeg_data'), 'ep_extract', 'famvideo_extract', 'eeglook','videolook')
