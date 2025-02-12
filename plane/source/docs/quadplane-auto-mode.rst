.. _quadplane-auto-mode:

QuadPlane AUTO Missions
=======================

You can ask the QuadPlane code to fly :ref:`AUTO <auto-mode>`
missions, with everything from automatic vertical takeoff, to mixtures
of fixed wing and VTOL waypoints and automated VTOL landings. The
result is an extremely versatile aircraft capable of long range
missions with vertical takeoff and landing.

.. image:: ../images/quadplane-vtol-mission.jpg
    :target: ../_images/quadplane-vtol-mission.jpg

AUTO VTOL Takeoff
-----------------

The most common use of VTOL mission commands in a QuadPlane is an
automatic VTOL takeoff. To use a VTOL takeoff you plan your auto
mission as usual with your ground stations mission editor, but instead
of a NAV_TAKEOFF command for a fixed wing takeoff you instead use a
NAV_VTOL_TAKEOFF command for a VTOL takeoff.

The only parameter to a NAV_VTOL_TAKEOFF is the altitude above the
takeoff point where the takeoff is complete. When executed, the aircraft will
start climbing at :ref:`Q_WP_SPEED_UP<Q_WP_SPEED_UP>`. Once that altitude is
reached the aircraft will move to the next waypoint, transitioning to
fixed wing flight as needed. The latitude and longitude of the
NAV_VTOL_TAKEOFF command is ignored.

If a NAV_VTOL_TAKEOFF is executed when already flying, the vehicle will enter VTOL mode and climb the amount given in the altitude parameter, above its current altitude. This behavior can be modified by using the "respect takeoff frames" bit of the :ref:`Q_OPTIONS<Q_OPTIONS>` bitmask which will skip to the next command if already at or above the altitude set in the parameter, or climb to it.

AUTO VTOL Landing
-----------------

There are several ways to perform an automatic VTOL landing. The
simplest is to include a NAV_VTOL_LAND command in your mission. That
command should use an altitude of zero, and have a latitude and
longitude of the landing position.

Versions prior to 4.1
~~~~~~~~~~~~~~~~~~~~~

In firmware version prior to 4.1 ,when using NAV_VTOL_LAND it is important to have the right horizontal
spacing between that waypoint and the previous one. As soon as the
aircraft starts on the NAV_VTOL_LAND waypoint it will transition to
VTOL flight, which means it will start flying much more slowly than it
does in fixed wing flight. So you need to put the previous waypoint
the right distance from the landing point. If it is too far from the
landing point then the aircraft will spend a lot of time in VTOL
flight which will waste battery. If it is too close to the landing
point then it will have to stop very abruptly in order to land.

For most small QuadPlanes a distance of between 60 and 80 meters from
the last waypoint to the landing point is good. For larger faster
flying QuadPlanes you will need a larger distance.

Also make sure the altitude of the last waypoint is chosen to be
within a reasonable height of the landing. The VTOL landing approach
will be done at whatever height the aircraft is at when it starts on
the NAV_VTOL_LAND waypoint. So you would typically want the previous
waypoint to have an altitude of about 20 meters above the ground.

If :ref:`Q_OPTIONS<Q_OPTIONS>` is set for "Use a fixed wing approach" (bit 4), then instead of transitioning to VTOL flight and doing a VTOL landing, it will remain in plane mode , and proceed to the landing position, climbing or descending to the altitude set in the NAV_VTOL_LAND waypoint. When it reaches within :ref:`Q_FW_LND_APR_RAD<Q_FW_LND_APR_RAD>` of the landing location, it will perform a LOITER_TO_ALT to finish the climb or descent to that altitude set in the waypoint and using :ref:`Q_FW_LND_APR_RAD<Q_FW_LND_APR_RAD>` for the loiter radius , then, turning into the wind, transition to VTOL mode and proceed to the landing location and land.

Note that if :ref:`Q_FW_LND_APR_RAD<Q_FW_LND_APR_RAD>` = 0, the waypoint loiter radius will be used. Be sure to set the VTOL_LAND waypoint altitude and :ref:`Q_FW_LND_APR_RAD<Q_FW_LND_APR_RAD>` distance to avoid obstacles. Note also, that if this waypoint is reached while in VTOL mode, it will proceed as a normal VTOL_LAND command, even if the option is set.

.. note:: Be sure that the NAV_VTOL_LAND is placed with enough distance to reach the :ref:`Q_FW_LND_APR_RAD<Q_FW_LND_APR_RAD>`, plus at least a 90 degree turn, plus distance required to de-transition to VTOL mode.

.. image:: ../images/VTOL_LAND.jpg
    :target: ../_images/VTOL_LAND.jpg

(figure courtesy of SRP AERO   https://srp.aero/)

Versions 4.1 and later
~~~~~~~~~~~~~~~~~~~~~~

By default a NAV_VTOL_LAND will remain in fixed mode at the current altitude, until close to the landing point, execute an "airbraking" maneuver to reduce speed, then transition to VTOL, navigate precisely to the landing point, and descend as in QLAND to a landing. This allows setting the NAV_VTOL_LAND point any distant away from the last waypoint without having to worry about excessive time in VTOL mode.

If :ref:`Q_OPTIONS<Q_OPTIONS>` bit 16 is set to disable the fixed wing approach phase, then it will immediately transition to VTOL when the command is executed and navigate to the landing point in VTOL mode. This requires the careful setup of the last waypoint before this command to avoid a long path to the landing point while in VTOL mode.

If :ref:`Q_OPTIONS<Q_OPTIONS>` bit 4 is set, then even if bit 16 is set, the vehicle will execute the fixed wing approach and loiter to altitude, described above in the previous section, before changing to VTOL mode. If bit 16 is also set, then when the vehicle switches to VTOL mode, it will attempt to do the airbrake maneuver and QLAND for that final sement.

Return to Launch
----------------

An alternaive to using a NAV_VTOL_LAND command is to use a
RETURN_TO_LAUNCH command, and to set the :ref:`Q_RTL_MODE<Q_RTL_MODE>` parameter to 1.

The advantage of using a RETURN_TO_LAUNCH with :ref:`Q_RTL_MODE<Q_RTL_MODE>` set is that
the aircraft will automatically use fixed wing flight until it gets
within :ref:`RTL_RADIUS <RTL_RADIUS>` of the return point. That makes
it easier to plan missions with a VTOL landing from anywhere in the
flying area.

.. image:: ../images/quadplane_RTL.jpg
    :target: ../_images/quadplane_RTL.jpg

Mixing VTOL and Fixed Wing Flight
---------------------------------

To mix fixed wing and VTOL flight in one mission you can use the
DO_VTOL_TRANSITION command in your mission. A DO_VTOL_TRANSITION
command takes a single parameter. If the parameter is set to 3 then
the aircraft will change to VTOL mode. If the parameter is set to 4
then it will change to fixed wing mode.

.. image:: ../images/quadplane-vtol-transition.jpg
    :target: ../_images/quadplane-vtol-transition.jpg

In the above example the aircraft will do a VTOL takeoff, then it will
fly to waypoint 1 as a fixed wing aircraft. It will then switch to
VTOL mode and fly as a VTOL aircraft through waypoints 4 and 5, then
it will switch back to fixed wing flight to reach waypoint 7, before
finally flying home and landing as a VTOL aircraft (assuming
Q_RTL_MODE is set to 1).

Hovering in a Mission
---------------------

By setting the :ref:`Q_GUIDED_MODE <Q_GUIDED_MODE>` parameter to 1
your quadplane will handle loiter commands in :ref:`GUIDED mode
<guided-mode>` and in AUTO missions as a VTOL aircraft. For example, the
following mission:

.. image:: ../images/quadplane-loiter-time.jpg
    :target: ../_images/quadplane-loiter-time.jpg

the aircraft will pause while hovering for 10 seconds at
waypoint 3. It will fly the rest of the mission as a fixed wing
aircraft. This can be very useful for getting good photographs of a
number of locations in a mission while flying most of the mission as
an efficient fixed wing aircraft.

Guided Mode
===========
             
In addition to AUTO mode, you can also use a QuadPlane in :ref:`GUIDED
mode <guided-mode>`. To use VTOL support in GUIDED mode you need to
set the :ref:`Q_GUIDED_MODE <Q_GUIDED_MODE>` parameter to 1. When set,
GUIDED mode behaviour will change so that the position hold at the
destination will be done as a VTOL hover rather than a fixed wing
circle.

The approach to the guided waypoint will be done as a fixed wing
aircraft. The transition to VTOL flight will begin at the
:ref:`WP_LOITER_RAD <WP_LOITER_RAD>` radius in meters. This should be
set appropriately for your aircraft. A value of 80 meters is good
for a wide range of QuadPlanes.

When hovering at the destination in GUIDED mode if a new GUIDED
destination is given then the aircraft will transition back to fixed
wing flight, fly to the new location and then hover again in VTOL
mode.

