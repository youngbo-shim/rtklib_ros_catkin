cmake_minimum_required(VERSION 2.8.3)
project(rtklibros)

#set(ROS_BUILD_TYPE Debug)

link_libraries(m rt pthread)

find_package(catkin REQUIRED COMPONENTS roscpp roslib std_msgs geometry_msgs message_generation gps_common)

set(SRC ../../../src)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3 -ansi -pedantic -Wno-unused-but-set-variable -I$(SRC) -I.. -DTRACE -DENAGLO -DENAQZS -DENACMP -DENAGAL -DENAIRN -DNFREQ=3 -DSVR_REUSEADDR -g")

add_definitions(-DENAGLO -DENAQZS -DENACMP -DENAGAL -DENAIRN -DNFREQ=3 -DSVR_REUSEADDR)

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

add_service_files(
  FILES
  SingleResponse.srv
)

generate_messages(DEPENDENCIES std_msgs)

catkin_package(CATKIN_DEPENDS std_msgs message_runtime)

include_directories (  
  include
  include/rtklibros
  ${catkin_INCLUDE_DIRS}
)

#pos2kml
add_library(convkml src/convkml.c)
add_library(solution src/solution.c)
add_library(geoid src/geoid.c)
add_library(rtkcmn src/rtkcmn.c)
add_library(convgpx src/convgpx.c)

add_executable(pos2kml app/pos2kml/pos2kml.c)
target_link_libraries(pos2kml convkml solution geoid rtkcmn solution convgpx)

#str2str
add_library(ss2 src/rcv/ss2.c)
add_library(crescent src/rcv/crescent.c)
add_library(skytraq src/rcv/skytraq.c)
add_library(gw10 src/rcv/gw10.c)
add_library(javad src/rcv/javad.c)
add_library(stream src/stream.c)
add_library(sbas src/sbas.c)
add_library(cmr src/rcv/cmr.c)
add_library(nvs src/rcv/nvs.c)
add_library(rt17 src/rcv/rt17.c)
add_library(tersus src/rcv/tersus.c)
add_library(septentrio src/rcv/septentrio.c)
add_library(binex src/rcv/binex.c)
add_library(ublox src/rcv/ublox.c src/rcvraw.c)
target_link_libraries(ublox ss2 crescent gw10 javad skytraq crescent sbas novatel rtkcmn rinex rtcm cmr nvs rt17 tersus septentrio binex)
add_library(novatel src/rcv/novatel.c)
add_library(streamsvr src/streamsvr.c)

add_executable(str2str app/str2str/str2str.c)
target_link_libraries(str2str stream rtkcmn solution geoid ublox skytraq streamsvr)
target_link_libraries(str2str ${catkin_LIBRARIES})

#rnx2rtkp

add_library(rinex src/rinex.c)
add_library(tides src/tides.c)
add_library(rtkpos src/rtkpos.c)
add_library(ephemeris src/ephemeris.c src/preceph.c)
target_link_libraries(rtkpos lambda ppp pntpos ephemeris tides)
add_library(qzslex src/qzslex.c)
add_library(postpos src/postpos.c)
add_library(ppp_corr src/ppp_corr.c)
target_link_libraries(postpos sbas rtkpos rtcm rinex qzslex ppp_corr)

add_library(lambda src/lambda.c)
add_library(preceph src/preceph.c)
add_library(ionex src/ionex.c)
add_library(pntpos src/pntpos.c)
target_link_libraries(pntpos ionex qzslex)
add_library(options src/options.c)
add_library(ppp_ar src/ppp_ar.c)
add_library(ppp src/ppp.c)
target_link_libraries(ppp pntpos ppp_ar ppp_corr)
add_library(rtcm src/rtcm.c)
add_library(rtcm2 src/rtcm2.c)
add_library(rtcm3 src/rtcm3.c)
add_library(rtcm3e src/rtcm3e.c)
target_link_libraries(rtcm rtcm2 rtcm3 rtcm3e)

add_executable(rnx2rtkp app/rnx2rtkp/rnx2rtkp.c)
target_link_libraries(rnx2rtkp rtkcmn postpos solution ionex geoid preceph pntpos ephemeris options rtcm ionex)

#convbin
add_library(convrnx src/convrnx.c)
target_link_libraries(convrnx rtkcmn sbas rtkpos rtcm rinex qzslex)

add_executable(convbin app/convbin/convbin.c src/rcvraw.c)
target_link_libraries(convbin ublox rtkcmn rinex sbas convrnx rtcm skytraq rtkcmn rtksvr rtkpos geoid solution lambda sbas stream rtcm preceph options pntpos ppp ublox ephemeris rinex ionex)
	
#rtkrcv

add_library(vt app/rtkrcv/vt.c)
add_library(rtksvr src/rtksvr.cc)
add_executable(rtkrcv app/rtkrcv/rtkrcvros.cc)
target_link_libraries(rtkrcv rtkcmn rtksvr rtkpos geoid solution lambda sbas stream rtcm rtcm2 rtcm3 rtcm3e preceph options pntpos ppp ppp_ar novatel ublox ss2 crescent skytraq gw10 javad nvs binex rt17 ephemeris rinex ionex qzslex tides ppp_corr septentrio cmr tersus vt)
target_link_libraries(rtksvr ${catkin_LIBRARIES} qzslex)
