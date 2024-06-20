# 22wh5a0401
# Classic Retro Snake Game

function snake(cmd)
%SNAKE  Graphical User Interface for playing "Nokia Classic Game" Snake.
%
%   The Game:
%   Make the snake grow longer by directing it to the food.
%   Extra bonus points are given from hearts, eat them as fast as possible
%   as amount of bonus points decreases with time.
%   Higher level gives more points for the food but will also make it
%   tougher to steer a long snake as speed increaces.
%   Five different mazes as well as play w/o maze are possible.
%
%   Game Controls:
%   Steer snake using arrow keys. Pause game with 'p' or space.
%
%   Example:
%       snake    % Start Main Snake Interface
%   Developed by Per-Anders Ekstr m, 2003-2007 Facilia AB.
global SNAKE MAZE LEVEL BOARD DIRECTION RUNNING FOOD SOUND BONUS PAUSE
if ~nargin
    cmd = 'init';
end
if ~(ischar(cmd)||isscalar(cmd))
    return;
end
switch cmd
    case 'init'
        scrsz = get(0,'ScreenSize');
        % Initialize figure window
        f = figure('Name','Snake',...
            'Numbertitle','off',...
            'Menubar','none',...
            'Color',[.95 .95 .95],...
            'DoubleBuffer','on',...
            'Position',[(scrsz(3)-400)/2 (scrsz(4)-300)/2 400 300],...
            'Colormap',[.58 .71 .65;.4 .4 .4;0 0 0;1 1 0],...
            'CloseRequestFcn',sprintf('%s(''Stop'');closereq;',mfilename),...
            'KeyPressFcn',sprintf('%s(double(get(gcbf,''Currentcharacter'')))',mfilename));
        % Create the Menu
        FileMenu = uimenu(f,'Label','&File');
        uimenu(FileMenu,'Label','New Game','Accelerator','N','Callback',sprintf('%s(''NewGame'')',mfilename));
        uimenu(FileMenu,'Label','Pause','Accelerator','N','Callback',sprintf('%s(''Pause'')',mfilename));
        uimenu(FileMenu,'Label','Exit','Accelerator','Q','Separator','on','Callback',sprintf('%s(''Stop'');closereq',mfilename));
        LevelMenu = uimenu(f,'Label','&Level');
        uimenu(LevelMenu,'Label','|','Callback',sprintf('%s(''Level'')',mfilename))
        uimenu(LevelMenu,'Label','||','Callback',sprintf('%s(''Level'')',mfilename))
        uimenu(LevelMenu,'Label','|||','Callback',sprintf('%s(''Level'')',mfilename))
        uimenu(LevelMenu,'Label','||||','Callback',sprintf('%s(''Level'')',mfilename))
        uimenu(LevelMenu,'Label','|||||','Callback',sprintf('%s(''Level'')',mfilename),'checked','on')
        uimenu(LevelMenu,'Label','||||||','Callback',sprintf('%s(''Level'')',mfilename))
        uimenu(LevelMenu,'Label','|||||||','Callback',sprintf('%s(''Level'')',mfilename))
        uimenu(LevelMenu,'Label','||||||||','Callback',sprintf('%s(''Level'')',mfilename))
        uimenu(LevelMenu,'Label','|||||||||','Callback',sprintf('%s(''Level'')',mfilename))
        MazesMenu = uimenu(f,'Label','&Mazes');
        uimenu(MazesMenu,'Label','No maze','Callback',sprintf('%s(''Mazes'')',mfilename),'checked','on')
        uimenu(MazesMenu,'Label','Box','Callback',sprintf('%s(''Mazes'')',mfilename))
        uimenu(MazesMenu,'Label','Tunnel','Callback',sprintf('%s(''Mazes'')',mfilename))
        uimenu(MazesMenu,'Label','Spiral','Callback',sprintf('%s(''Mazes'')',mfilename))
        uimenu(MazesMenu,'Label','Blockade','Callback',sprintf('%s(''Mazes'')',mfilename))
        uimenu(MazesMenu,'Label','Twisted','Callback',sprintf('%s(''Mazes'')',mfilename))
        HelpMenu = uimenu(f,'Label','&Help');
        uimenu(HelpMenu,'Label','Help','Callback',sprintf('helpwin %s',mfilename))
        uimenu(HelpMenu,'Label','Show Score','Callback',sprintf('%s(''ShowScore'')',mfilename),'Separator','on','Checked','on')
        uimenu(HelpMenu,'Label','Play Sounds','Callback',sprintf('%s(''PlaySound'')',mfilename),'Checked','on')
        uimenu(HelpMenu,'Label','About','Callback',sprintf('%s(''About'')',mfilename),'Separator','on');
        % Create The axes
        axes('Units','normalized',...
            'Position', [0 0 1 1],...
            'Visible','off',...
            'DrawMode','fast',...
            'NextPlot','replace');
        % Add the board
        BOARD = image(getTitle,'CDataMapping','scaled');
        axis image
        set(gca,...
            'XTick',NaN,...
            'YTick',NaN)
        text(40,30,'0',...
            'FontUnits','normalized', ...
            'FontSize',0.03, ...
            'FontName','FixedWidth',...
            'FontWeight','bold',...
            'Color',[1 1 1],...
            'VerticalAlignment','baseline', ...
            'HorizontalAlignment','right',...
            'Tag','Score');
        SNAKE = [14,20;14,19;14,18;14,17;14,16];
        MAZE = zeros(30,40);
        LEVEL = 4;
        SOUND = true;
    case 28 % left
        if SNAKE(2,2)~=mod(SNAKE(1,2)-2,40)+1
            DIRECTION = cmd;
        end
    case 29 % right
        if SNAKE(2,2)~=mod(SNAKE(1,2),40)+1
            DIRECTION = cmd;
        end
    case 30 % up
        if SNAKE(2,1)~=mod(SNAKE(1,1)-2,30)+1
            DIRECTION = cmd;
        end
    case 31 % down
        if SNAKE(2,1)~=mod(SNAKE(1,1),30)+1
            DIRECTION = cmd;
        end
    case 'Level' % Change of Level
        set(get(get(gcbo,'Parent'),'Children'),'checked','off')
        set(gcbo,'checked','on')
        LEVEL = length(get(gcbo,'Label'));
    case 'Mazes' % Change of Maze
        set(get(get(gcbo,'Parent'),'Children'),'checked','off')
        set(gcbo,'checked','on')
        MAZE = zeros(30,40);
        switch get(gcbo,'Label')
            case 'No maze'
            case 'Box'
                MAZE([1 30],:) = 1;
                MAZE(:,[1 40]) = 1;
            case 'Tunnel'
                MAZE([1:4 26:30],[1 40]) = 1;
                MAZE([1 30],[1:4 36:40]) = 1;
                MAZE([10 20],15:25) = 1;
            case 'Spiral'
                MAZE(1:15,15) = 1;
                MAZE(15:30,25) = 1;
                MAZE(25,1:15) = 1;
                MAZE(5,25:40) = 1;
            case 'Blockade'
                MAZE([1:10 end-10:end],[1 end]) = 1;
                MAZE([1 end],:) = 1;
                MAZE([8 22],15:25) = 1;
                MAZE(5:10,[10 30]) = 1;
                MAZE(20:25,[10 30]) = 1;
            case 'Twisted'
                MAZE([1 2 3 9 16 31 39 46 61 69 76 91 99 106 121 129    ...
                    136 159 166 189 196 219 226 249 256 279 286 301 309 ...
                    316 331 339 346 361 369 376 391 399 406 421 429 436 ...
                    451 459 466 481 489 496 511 512 513 514 515 516 517 ...
                    518 519 526 541 556 571 586 601 616 631 646 661 676 ...
                    677 678 679 680 681 682 683 684 685 686 687 688 689 ...
                    690 691 699 706 721 729 736 751 759 766 781 789 796 ...
                    811 819 826 841 849 856 871 879 886 901 909 916 931 ...
                    939 946 969 976 999 1006 1029 1036 1059 1066 1089 ...
                    1096 1119 1126 1149 1156 1179 1186]) = 1;
        end
        feval(mfilename,'Stop')
    case 'ShowScore' % Change of Show Score
        switch get(gcbo,'checked')
            case 'on'
                set(gcbo,'checked','off')
                set(findobj(gcbf,'Tag','Score'),'Visible','off')
            case 'off'
                set(gcbo,'checked','on')
                set(findobj(gcbf,'Tag','Score'),'Visible','on')
        end
    case 'PlaySound' % Change of Play Sounds
        switch get(gcbo,'checked')
            case 'on'
                set(gcbo,'checked','off')
                SOUND = false;
            case 'off'
                set(gcbo,'checked','on')
                SOUND = true;
        end
    case 'NewGame' % New Game N or Ctrl-N
        set(findobj(gcbf,'Tag','Score'),'String','0')
        SNAKE = [14,20;14,19;14,18;14,17;14,16];
        DIRECTION = 29; % right
        BONUS = 0;
        PAUSE = false;
        feval(mfilename,'Food')
        feval(mfilename,'Start')
    case 'Start' % Start Game
        RUNNING = true;
        bonusCounter = 0;
        foodCounter = 0;
        while(RUNNING)
            if ~PAUSE
                SNAKE = circshift(SNAKE,1);
                SNAKE(1,:) = SNAKE(2,:);
                switch DIRECTION
                    case 28 % left
                        SNAKE(1,2) = mod(SNAKE(1,2)-2,40)+1;
                    case 29 % right
                        SNAKE(1,2) = mod(SNAKE(1,2),40)+1;
                    case 30 % up
                        SNAKE(1,1) = mod(SNAKE(1,1)-2,30)+1;
                    case 31 % down
                        SNAKE(1,1) = mod(SNAKE(1,1),30)+1;
                end
                % Check if snake hits any barrier
                if MAZE(SNAKE(1,1),SNAKE(1,2)) || ...
                        sum(ismember(SNAKE(2:end,1),SNAKE(1,1))+...
                        ismember(SNAKE(2:end,2),SNAKE(1,2))==2)
                    %If play sound
                    if SOUND
                        soundsc(sin(1:100),1000)
                    end
                    pause(.3)
                    delete(findobj(gcbf,'Tag','Bonus'))
                    feval(mfilename,'Stop')
                    set(BOARD,'CData',getGameOver)
                else
                    % Check if snake eats bonus
                    if isequal(SNAKE(1,:),BONUS)
                        % Update score
                        scorehandle = findobj(gcbf,'Tag','Score');
                        set(scorehandle,'String',...
                            num2str(LEVEL*ceil(bonusCounter/3)+...
                            str2double(get(scorehandle,'String'))))
                        %If play sound
                        if SOUND
                            soundsc(sin(1:100),5000)
                        end
                        bonusCounter = 1;
                    end
                    if BONUS
                        bonusCounter = bonusCounter-1;
                        if bonusCounter<=0
                            delete(findobj(gcbf,'Tag','Bonus'))
                            BONUS = 0;
                        end
                    end
                    % Check if snake eats food
                    if isequal(SNAKE(1,:),FOOD)
                        % Snake Grows!
                        SNAKE(end+1,:) = SNAKE(end,:);
                        % Update score
                        scorehandle = findobj(gcbf,'Tag','Score');
                        set(scorehandle,'String',...
                            num2str(LEVEL+str2double(get(scorehandle,'String'))))
                        %If play sound
                        if SOUND
                            soundsc(sin(1:100),10000)
                        end
                        % Spawn new food
                        feval(mfilename,'Food')
                        if ~BONUS % only update if no bonus exist
                            bonusCounter = bonusCounter+15;
                            foodCounter = foodCounter+1;
                        end
                        if foodCounter==4 % Spawn new bonus every fourth Food
                            feval(mfilename,'Bonus')
                            foodCounter = 0;
                        end
                    end
                    feval(mfilename,'DrawBoard')
                end
            end
            pause(.1/LEVEL)
        end
    case {112 32} % Pause Game
        PAUSE=~PAUSE;
        if PAUSE && RUNNING
            set(BOARD,'CData',getPause)
        end
    case 'Stop' % Stop Game
        RUNNING = false;
        set(BOARD,'CData',getTitle)
    case 'Food' % Put food onto game board
        CData = MAZE;
        for i=1:size(SNAKE,1)
            CData(SNAKE(i,1),SNAKE(i,2)) = 1;
        end
        ind = find(CData'==0);
        ind = ind(ceil(rand*length(ind)));
        FOOD =  [ceil(ind/40) mod(ind-1,40)+1];
    case 'Bonus' % Put bonus onto game board
        delete(findobj(gcbf,'Tag','Bonus'))
        CData = MAZE;
        for i=1:size(SNAKE,1)
            CData(SNAKE(i,1),SNAKE(i,2)) = 1;
        end
        CData(FOOD(1,1),FOOD(1,2)) = 1;
        ind = find(CData'==0);
        ind = ind(ceil(rand*length(ind)));
        BONUS =  [ceil(ind/40) mod(ind-1,40)+1];
        text(BONUS(2),BONUS(1),'\heartsuit',...
            'Color',[1 0 0],...
            'FontUnits','normalized',...
            'FontSize',.065,...
            'HorizontalAlignment','Center',...
            'VerticalAlignment','Middle',...
            'Tag','Bonus')
    case 'DrawBoard' % Draw the Game Board
        CData = MAZE;
        for i=1:size(SNAKE,1)
            CData(SNAKE(i,1),SNAKE(i,2)) = 2;
        end
        CData(FOOD(1),FOOD(2)) = 4;
        set(BOARD,'CData',CData)
    case 'About'
        [ico,map] = getIcon();
        msgbox(sprintf([...
            'Graphical User Interface for playing "Nokia Classic Game" Snake.\n\n'...
            'Developed by Per-Anders Ekstr m, 2003-2007 Facilia AB\n'...
            'E-mail: peranders.ekstrom@facilia.se']),...
            'About Snake','custom',ico,map)
end

function [ico,map]=getIcon()
% create simple icon matrix
ico = ones(13)*3;
ico(:,1:4:13) = 1;
ico(1:4:13,:) = 1;
ico(6:8,6:8) = 2;
ico(6:8,10:12) = 2;
ico(10:12,10:12) = 2;
map = [0 0 0;.5 .5 .6;[148 182 166]/255;];
function title = getTitle()
title = zeros(30,40);
title([42 43 47 48 72 73 77 78 104 105 106 107 108 134 135 136 137 138 ...
    164 165 166 167 168 222 223 224 225 226 227 228 252 253 254 255 256 ...
    257 258 282 283 312 313 344 345 346 347 348 374 375 376 377 378 404 ...
    405 406 407 408 464 465 466 494 495 496 522 523 527 528 552 553 557 ...
    558 582 583 587 588 612 613 614 615 616 617 618 642 643 644 645 646 ...
    647 648 672 673 674 675 676 677 678 727 728 729 730 731 732 733 734 ...
    735 736 737 738 757 758 759 760 761 762 763 764 765 766 767 768 787 ...
    788 789 790 791 792 793 794 795 796 797 798 824 825 826 854 855 856 ...
    882 883 887 888 912 913 917 918 972 973 974 975 976 977 978 1002 ...
    1003 1004 1005 1006 1007 1008 1032 1033 1037 1038 1062 1063 1067 ...
    1068 1092 1093 1097 1098 1122 1123 1124 1125 1126 1152 1153 1154 ...
    1155 1156]) = 3;
function gameover = getGameOver()
gameover = zeros(30,40);
gameover([95 96 97 98 99 100 101 102 103 104 109 110 111 112 113 114 ...
    125 126 127 128 129 130 131 132 133 134 139 140 141 142 143 144 155 ...
    156 163 164 167 168 175 176 185 186 193 194 197 198 205 206 215 216 ...
    219 220 221 222 223 224 229 230 231 232 233 234 245 246 249 250 251 ...
    252 253 254 259 260 261 262 263 264 335 336 337 338 339 340 341 342 ...
    343 344 347 348 349 350 351 352 353 354 365 366 367 368 369 370 371 ...
    372 373 374 377 378 379 380 381 382 383 384 395 396 399 400 415 416 ...
    425 426 429 430 445 446 455 456 457 458 459 460 461 462 463 464 467 ...
    468 469 470 471 472 473 474 485 486 487 488 489 490 491 492 493 494 ...
    497 498 499 500 501 502 503 504 575 576 577 578 579 580 581 582 583 ...
    584 587 588 589 590 591 592 593 594 595 596 605 606 607 608 609 610 ...
    611 612 613 614 617 618 619 620 621 622 623 624 625 626 635 636 647 ...
    648 651 652 655 656 665 666 677 678 681 682 685 686 695 696 697 698 ...
    699 700 701 702 703 704 707 708 711 712 715 716 725 726 727 728 729 ...
    730 731 732 733 734 737 738 741 742 745 746 755 756 785 786 817 818 ...
    819 820 821 822 823 824 827 828 829 830 831 832 833 834 835 836 847 ...
    848 849 850 851 852 853 854 857 858 859 860 861 862 863 864 865 866 ...
    887 888 891 892 917 918 921 922 935 936 937 938 939 940 941 942 943 ...
    944 949 950 953 954 955 956 965 966 967 968 969 970 971 972 973 974 ...
    979 980 983 984 985 986 995 996 999 1000 1003 1004 1025 1026 1029 ...
    1030 1033 1034 1055 1056 1059 1060 1063 1064 1067 1068 1069 1070 ...
    1071 1072 1075 1076 1085 1086 1093 1094 1097 1098 1099 1100 1101 ...
    1102 1105 1106]) = 1;
function pause = getPause()
pause = zeros(30,40);
pause([41 42 43 44 45 46 47 48 49 50 71 72 73 74 75 76 77 78 79 80 101 ...
    102 103 104 105 106 107 108 109 110 131 132 136 137 161 162 166 167 ...
    193 194 195 223 224 225 283 284 285 313 314 315 341 342 346 347 371 ...
    372 376 377 401 402 406 407 431 432 433 434 435 436 437 461 462 463 ...
    464 465 466 467 491 492 493 494 495 496 497 551 552 553 554 555 556 ...
    581 582 583 584 585 586 616 617 646 647 671 672 673 674 675 676 677 ...
    701 702 703 704 705 706 707 731 732 733 734 735 736 737 791 792 796 ...
    797 821 822 826 827 853 854 855 856 857 883 884 885 886 887 913 914 ...
    915 916 917 971 972 973 974 975 976 977 1001 1002 1003 1004 1005 ...
    1006 1007 1031 1032 1036 1037 1061 1062 1066 1067 1091 1092 1096 ...
    1097 1121 1122 1123 1124 1125 1151 1152 1153 1154 1155]) = 1;
