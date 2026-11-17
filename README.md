
const APPLE_MUSIC_MODULE = {
  id: 'apple-music-all-in-one',
  name: 'Apple Music All-in-One',
  author: 'Custom 8SPINE',
  version: '1.0.0',
  labels: ['APPLE MUSIC', 'METADATA', 'PREVIEW'],
  description: 'Apple Music catalog search, metadata, artwork and audio previews via iTunes API.',

  async searchTracks(query, limit = 25) {
    const url =
      'https://itunes.apple.com/search?term=' +
      encodeURIComponent(query) +
      '&entity=song&limit=' +
      Math.min(limit, 50) +
      '&country=id';

    const response = await fetch(url);
    if (!response.ok) throw new Error('Apple search failed');

    const data = await response.json();

    const tracks = (data.results || []).map(item => ({
      id: String(item.trackId),
      title: item.trackName || 'Unknown Title',
      artist: item.artistName || 'Unknown Artist',
      album: item.collectionName || '',
      duration: Math.floor((item.trackTimeMillis || 0) / 1000),
      albumCover: item.artworkUrl100
        ? item.artworkUrl100.replace('100x100bb', '600x600bb')
        : null,
      appleMusicUrl: item.trackViewUrl || null,
      previewUrl: item.previewUrl || null,
      source: 'Apple Music'
    }));

    return {
      tracks,
      total: data.resultCount || tracks.length
    };
  },

  async getTrackStreamUrl(trackId, quality) {
    const url =
      'https://itunes.apple.com/lookup?id=' +
      encodeURIComponent(trackId) +
      '&country=id';

    const response = await fetch(url);
    if (!response.ok) throw new Error('Track lookup failed');

    const data = await response.json();
    const item = (data.results || [])[0];

    if (!item || !item.previewUrl) {
      throw new Error('Audio preview unavailable');
    }

    return {
      streamUrl: item.previewUrl,
      track: {
        id: String(trackId),
        audioQuality: 'PREVIEW'
      }
    };
  },

  async getAlbum(albumId) {
    const url =
      'https://itunes.apple.com/lookup?id=' +
      encodeURIComponent(albumId) +
      '&entity=song&country=id';

    const response = await fetch(url);
    if (!response.ok) throw new Error('Album lookup failed');

    const data = await response.json();
    const items = data.results || [];
    const album = items[0];

    return {
      album: album ? {
        id: String(album.collectionId),
        title: album.collectionName,
        artist: album.artistName,
        artwork: album.artworkUrl100
      } : null,

      tracks: items
        .filter(item => item.wrapperType === 'track')
        .map(item => ({
          id: String(item.trackId),
          title: item.trackName,
          artist: item.artistName,
          album: item.collectionName,
          duration: Math.floor((item.trackTimeMillis || 0) / 1000),
          albumCover: item.artworkUrl100,
          previewUrl: item.previewUrl
        }))
    };
  }
};

return APPLE_MUSIC_MODULE;
