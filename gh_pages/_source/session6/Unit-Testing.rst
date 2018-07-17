Unit Testing
============

In this exercise we will write a unit tests in the myworkcell_core package.

Motivation
----------

The ROS Scan-N-Plan application is complete and documented.  Now we want to test the program to make sure it behaves as expected.

Information and Resources
-------------------------

`Google Test <https://github.com/google/googletest>`__: C++ XUnit test framework

`rostest <http://wiki.ros.org/rostest>`__: ROS wrapper for XUnit test framework

`catkin testing <http://catkin-tools.readthedocs.io/en/latest/verbs/catkin_build.html?highlight=run_tests#building-and-running-tests>`__: Building and running tests with catkin

Problem Statement
-----------------

We have completed and and documented our Scan-N-Plan program.  We need to create a test framework so we can be sure our program runs as intended after it is built. In addition to ensuring the code runs as intended, unit tests allow you to easily check if new code changes functionality in unexpected ways.  Your goal is to create the unit test frame work and write a few tests. 

Guidance
--------

Create the unit test frame work
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Create a `test` folder in the myworkcell_core/src folder. In the workspace directory:

   .. code-block:: bash

	       catkin build
	       source devel/setup.bash
	       roscd myworkcell_core
	       mkdir src/test

#. Create utest.cpp file in the `myworkcell_core/src/test` folder:

   .. code-block:: bash

	       touch src/test/utest.cpp

#. Open utest.cpp in QT and include ros & gtest:

   .. code-block:: c++

	       #include <ros/ros.h>
	       #include <gtest/gtest.h>

#. Write a dummy test that will return true if executed. This will test our framework and we will replace it later with more useful tests:

   .. code-block:: c++

            TEST(TestSuite, myworkcell_core_framework)
            {
              ASSERT_TRUE(true);
            }

#. Next include the general main function, which will execute the unit tests we write later:

   .. code-block:: c++

            int main(int argc, char **argv)
            {
              testing::InitGoogleTest(&argc, argv);
              return RUN_ALL_TESTS();
            }

#. Edit myworkcell_core CMakeLists.txt to build the u_test.cpp file.  Append CMakeLists.txt:

   .. code-block:: cmake

	       if(CATKIN_ENABLE_TESTING)
		      find_package(rostest REQUIRED)
		      add_rostest_gtest(utest_node test/utest_launch.test src/test/utest.cpp)
		      target_link_libraries(utest_node ${catkin_LIBRARIES})
	       endif()

#. Create a test folder under myworkcell_core

   .. code-block:: bash

            mkdir test

#. Create a test launch file:

   .. code-block:: bash

	       touch test/utest_launch.test

#. Open the utest_launch.test file in QT and populate the file:

   .. code-block:: xml

            <?xml version="1.0"?>
            <launch>
                <node pkg="fake_ar_publisher" type="fake_ar_publisher_node" name="fake_ar_publisher"/>
                <test test-name="unit_test_node" pkg="myworkcell_core" type="utest_node"/>
            </launch>

#. Test the framework

   .. code-block:: bash

	       catkin build
	       catkin run_tests

   The console output should show:

   .. code-block:: bash

            [ROSTEST]-----------------------------------------------------------------------

            [myworkcell_core.rosunit-unit_test_node/myworkcell_core_framework][passed]

            SUMMARY
             * RESULT: SUCCESS
             * TESTS: 1
             * ERRORS: 0
             * FAILURES: 0

   This means our framework is functional and now we can add usefull unit tests.

Add stock publisher tests
^^^^^^^^^^^^^^^^^^^^^^^^^

#. The rostest package provides several tools for inspecting basic topic characteristics `hztest <http://wiki.ros.org/rostest/Nodes#hztest>`__, `paramtest <http://wiki.ros.org/rostest/Nodes#paramtest>`__, `publishtest <http://wiki.ros.org/rostest/Nodes#publishtest>`__.  We'll add some basic tests to verify that the `fake_ar_publisher` node is outputting the expected topics.

#. Add the test description to the `utest_launch.test` file:

   .. code-block:: xml

            <test name="publishtest" test-name="publishtest" pkg="rostest" type="publishtest">
                <rosparam>
                  topics:
                    - name: "/ar_pose_marker"
                      timeout: 10
                      negative: False
                    - name: "/ar_pose_visual"
                      timeout: 10
                      negative: False
                </rosparam>
            </test>

#. Run the test:

   .. code-block:: xml

            rostest myworkcell_core utest_launch.test

You should see:

	Summary: 2 tests, 0 errors, 0 failures

Write specific unit tests
^^^^^^^^^^^^^^^^^^^^^^^^^

#. Since we will be testing the messages we get from the fake_ar_publisher package, include the relevant header file (in `utest.cpp`):

   .. code-block:: c++

	       #include <fake_ar_publisher/ARMarker.h>

#. Declare a global variable:

   .. code-block:: c++

	       fake_ar_publisher::ARMarkerConstPtr test_msg_;

#. Add a subscriber callback to copy incoming messages to the global variable:

   .. code-block:: c++

            void testCallback(const fake_ar_publisher::ARMarkerConstPtr &msg)
            {
              test_msg_ = msg;
            }

#. Write a unit test to check the reference frame of the ar_pose_marker:

   .. code-block:: c++

            TEST(TestSuite, myworkcell_core_fake_ar_pub_ref_frame)
            {
                ros::NodeHandle nh;
                ros::Subscriber sub = nh.subscribe("/ar_pose_marker", 1, &testCallback);

                EXPECT_NE(ros::topic::waitForMessage<fake_ar_publisher::ARMarker>("/ar_pose_marker", ros::Duration(10)), nullptr);
                EXPECT_EQ(1, sub.getNumPublishers());
                EXPECT_EQ(test_msg_->header.frame_id, "camera_frame");
            }

#. Run the test:

   .. code-block:: bash

	       catkin build
	       rostest myworkcell_core utest_launch.test

#. view the results of the test:

   .. code-block:: bash

	       catkin_test_results build/myworkcell_core

You should see:

	Summary: 3 tests, 0 errors, 0 failures
