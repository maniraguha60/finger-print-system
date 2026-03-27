function getDistance(lat1, lon1, lat2, lon2) {
  const R = 6371; // Earth radius (km)

  const dLat = (lat2 - lat1) * Math.PI / 180;
  const dLon = (lon2 - lon1) * Math.PI / 180;

  const a =
    Math.sin(dLat/2) * Math.sin(dLat/2) +
    Math.cos(lat1 * Math.PI / 180) *
    Math.cos(lat2 * Math.PI / 180) *
    Math.sin(dLon/2) * Math.sin(dLon/2);

  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

  return R * c; // distance in km
}

module.exports = getDistance;const getDistance = require("../config/distance");

// GET buses near user with ETA
router.post("/nearby", async (req, res) => {
  const { userLat, userLng } = req.body;

  const buses = await Bus.find();

  const result = buses.map(bus => {
    const distance = getDistance(
      userLat,
      userLng,
      bus.latitude,
      bus.longitude
    );

    const speed = 40; // km/h (average)
    const timeHours = distance / speed;
    const timeMinutes = Math.round(timeHours * 60);

    return {
      ...bus._doc,
      distance: distance.toFixed(2),
      eta: timeMinutes
    };
  });

  res.json(result);
});
